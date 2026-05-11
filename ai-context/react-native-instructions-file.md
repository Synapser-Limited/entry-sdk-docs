---
layout: default
title: React Native Instructions File
nav_order: 4
parent: AI Integration Context
---

# Entry React Native — AI Integration Context

## What is Entry?

Entry is a **biometric identity verification SDK**. It is a **UI component** — not a raw API. You call `identifyUser()`, the SDK presents its own liveness and face-matching UI, and it returns a user object or throws an error. Do not call the Entry API directly.

## Key facts

- Entry does not have a native React Native SDK — you bridge to the iOS and Android native SDKs
- The bridge uses React Native's `NativeModules` system (Objective-C macros for iOS, `ReactContextBaseJavaModule` for Android)
- Requires a **physical device** — the iOS Simulator and Android Emulator are not supported
- **Expo Go does not support native modules** — you must use a dev build or bare workflow
- Errors cross the bridge as string codes — map them back to display messages in JS
- The JS bridge interface is identical for iOS and Android — you write one set of JS, two native implementations

## Before you can use Entry

Two prerequisites — done by the app owner, not the integrating developer:
1. Both the iOS **bundle identifier** and Android **package name** must be registered with the Entry team
2. Each developer's **GitHub account** must be invited to the `synapser-sdk-distribution` org

Contact support@synapser.com for both. Accept the GitHub org invitation before trying to install either SDK.

## Environment requirements

| Requirement | Minimum |
|---|---|
| OS (iOS target) | **macOS only** — building the iOS side requires Xcode |
| OS (Android target) | macOS, Windows, or Linux |
| Node.js | 18+ |
| React Native | 0.71+ |
| Xcode | 14+ (iOS target) |
| Android Studio | Hedgehog (2023.1) or later (Android target) |
| JDK | 11+ |
| iOS deployment target | 14.0+ |
| Android minSdk | 21 (Android 5.0) |
| Device | Physical device required for both iOS and Android — simulators/emulators not supported |
| Expo Go | Not supported — use a dev build (`npx expo run:ios`) or bare workflow |

---

## Architecture

```
React Native JS
      │
      ▼
NativeModules.EntryBridgeModule
      │
      ├── iOS  → EntrySDKClient (Swift, wraps WKWebView)
      └── Android → EntrySDK (Kotlin, wraps WebView)
```

You write one native bridge module per platform. Both modules export the same method names. The JS layer calls them via `NativeModules` without knowing which platform it's on.

---

## Step 1 — Install the native SDKs

### iOS — Swift Package Manager

1. Open `ios/YourApp.xcworkspace` in Xcode
2. **File → Add Package Dependencies…**
3. Enter the SPM URL provided by the Entry team
4. Rule: **Up to Next Major Version**
5. Add the `EntrySDK` library to your app target

You must be signed into GitHub in **Xcode → Settings → Accounts** with your invited account.

### Android — Gradle / Maven

Add GitHub credentials to `~/.gradle/gradle.properties` (never in version control):

```properties
github.username=YOUR_GITHUB_USERNAME
github.token=YOUR_GITHUB_PAT
```

Your PAT needs `read:packages` scope.

In `android/settings.gradle.kts`:
```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/synapser-sdk-distribution/entry-android-sdk")
            credentials {
                username = providers.gradleProperty("github.username").orNull
                    ?: System.getenv("GITHUB_USERNAME")
                password = providers.gradleProperty("github.token").orNull
                    ?: System.getenv("GITHUB_TOKEN")
            }
        }
    }
}
```

In `android/app/build.gradle.kts`:
```kotlin
dependencies {
    implementation("com.synapser:entry-sdk:<version>")
}
```

Version number is provided by the Entry team.

---

## Step 2 — Add permissions

### iOS — `ios/YourApp/Info.plist`

```xml
<key>NSCameraUsageDescription</key>
<string>Entry needs camera access for identity verification.</string>
```

### Android — `android/app/src/main/AndroidManifest.xml`

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="true" />
```

---

## Step 3 — Write the iOS bridge module

### `ios/YourApp/EntryBridgeModule.swift`

```swift
import Foundation
import React
import EntrySDK

@objc(EntryBridgeModule)
class EntryBridgeModule: NSObject {

    @objc static func requiresMainQueueSetup() -> Bool { true }

    @objc func configure(_ appName: String, environment: String) {
        let env: EntryEnvironment
        switch environment {
        case "live":  env = .live
        case "demo":  env = .demo
        case "test":  env = .test
        default:      env = .development
        }
        EntrySDKClient.shared.configure(appName: appName, environment: env)
    }

    @objc func identifyUser(
        _ registerIfNotFound: Bool,
        resolve: @escaping RCTPromiseResolveBlock,
        reject: @escaping RCTPromiseRejectBlock
    ) {
        DispatchQueue.main.async {
            guard let rootVC = UIApplication.shared.windows.first?.rootViewController else {
                reject("NO_PRESENTER", "No root view controller", nil)
                return
            }
            Task {
                do {
                    let user = try await EntrySDKClient.shared.identifyUser(
                        registerIfNotFound: registerIfNotFound,
                        presenter: rootVC
                    )
                    resolve([
                        "entryUserId":  user.entryUserId,
                        "firstName":    user.firstName,
                        "lastName":     user.lastName,
                        "emailAddress": user.emailAddress
                    ])
                } catch let error as EntrySDKError {
                    reject(error.code.rawValue, error.userMessage, error)
                } catch {
                    reject("UNKNOWN", error.localizedDescription, error)
                }
            }
        }
    }
}
```

### `ios/YourApp/EntryBridgeModule.m` (Objective-C macro file — required)

```objc
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(EntryBridgeModule, NSObject)

RCT_EXTERN_METHOD(
    configure:(NSString *)appName
    environment:(NSString *)environment
)

RCT_EXTERN_METHOD(
    identifyUser:(BOOL)registerIfNotFound
    resolve:(RCTPromiseResolveBlock)resolve
    reject:(RCTPromiseRejectBlock)reject
)

@end
```

---

## Step 4 — Write the Android bridge module

### `android/app/src/main/java/com/yourapp/entry/EntryBridgeModule.kt`

```kotlin
package com.yourapp.entry

import androidx.appcompat.app.AppCompatActivity
import com.facebook.react.bridge.*
import com.synapser.entry.EntryEnvironment
import com.synapser.entry.EntrySDK
import com.synapser.entry.models.EntrySDKError
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch

class EntryBridgeModule(reactContext: ReactApplicationContext) :
    ReactContextBaseJavaModule(reactContext) {

    override fun getName() = "EntryBridgeModule"

    @ReactMethod
    fun configure(appName: String, environment: String) {
        val env = when (environment) {
            "live"  -> EntryEnvironment.LIVE
            "demo"  -> EntryEnvironment.DEMO
            "test"  -> EntryEnvironment.TEST
            else    -> EntryEnvironment.DEVELOPMENT
        }
        EntrySDK.initialize(reactApplicationContext, appName, env)
    }

    @ReactMethod
    fun identifyUser(registerIfNotFound: Boolean, promise: Promise) {
        val activity = currentActivity as? AppCompatActivity ?: run {
            promise.reject("NO_ACTIVITY", "No AppCompatActivity available")
            return
        }
        CoroutineScope(Dispatchers.Main).launch {
            EntrySDK.getInstance().identifyUser(
                registerIfNotFound = registerIfNotFound,
                activity = activity
            )
            .onSuccess { user ->
                val map = Arguments.createMap().apply {
                    putString("entryUserId",  user.entryUserId)
                    putString("firstName",    user.firstName)
                    putString("lastName",     user.lastName)
                    putString("emailAddress", user.emailAddress)
                }
                promise.resolve(map)
            }
            .onFailure { error ->
                val sdkError = error as? EntrySDKError
                promise.reject(
                    sdkError?.code?.value ?: "UNKNOWN",
                    error.message ?: "An error occurred"
                )
            }
        }
    }
}
```

### `android/app/src/main/java/com/yourapp/entry/EntryBridgePackage.kt`

```kotlin
package com.yourapp.entry

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class EntryBridgePackage : ReactPackage {
    override fun createNativeModules(ctx: ReactApplicationContext): List<NativeModule> =
        listOf(EntryBridgeModule(ctx))
    override fun createViewManagers(ctx: ReactApplicationContext): List<ViewManager<*, *>> =
        emptyList()
}
```

### Register the package in `android/app/src/main/java/com/yourapp/MainApplication.kt`

```kotlin
override fun getPackages(): List<ReactPackage> =
    PackageList(this).packages.apply {
        add(EntryBridgePackage())
    }
```

---

## Step 5 — Write the JS interface

Create `src/entry/EntrySDK.ts`:

```typescript
import { NativeModules } from 'react-native';

const { EntryBridgeModule } = NativeModules;

export interface EntryUser {
  entryUserId: string;
  firstName: string;
  lastName: string;
  emailAddress: string;
}

export type EntryEnvironment = 'development' | 'test' | 'demo' | 'live';

/**
 * Configure Entry. Call once at app startup before calling identifyUser.
 */
export function configure(appName: string, environment: EntryEnvironment = 'test'): void {
  if (!EntryBridgeModule) {
    throw new Error(
      'EntryBridgeModule is not available. Expo Go is not supported — use a dev build.'
    );
  }
  EntryBridgeModule.configure(appName, environment);
}

/**
 * Identify (or register) a user via biometric liveness.
 * Presents the Entry UI modally over the current screen.
 */
export async function identifyUser(registerIfNotFound = true): Promise<EntryUser> {
  if (!EntryBridgeModule) {
    throw new Error(
      'EntryBridgeModule is not available. Expo Go is not supported — use a dev build.'
    );
  }
  return EntryBridgeModule.identifyUser(registerIfNotFound);
}
```

---

## Step 6 — Configure and call Entry

### Configure once at app startup (`App.tsx` or root component)

```tsx
import { useEffect } from 'react';
import { configure } from './entry/EntrySDK';

export default function App() {
  useEffect(() => {
    configure('your-app-name', 'test'); // switch to 'live' in production
  }, []);

  // ...
}
```

### Identify a user

```tsx
import { identifyUser, EntryUser } from './entry/EntrySDK';

async function handleLogin() {
  try {
    const user: EntryUser = await identifyUser(true);
    console.log('Identified:', user.entryUserId, user.firstName, user.lastName);
  } catch (error: any) {
    handleEntryError(error.code, error.message);
  }
}
```

### Error handling

Errors cross the bridge as `{ code: string, message: string }`. Map the codes to display messages:

```typescript
function handleEntryError(code: string, message: string) {
  switch (code) {
    case 'USER_NOT_FOUND':
      // User passed liveness but is not registered — prompt to register
      break;
    case 'LIVENESS_CHECK_FAILED':
      // Show lighting/positioning advice and offer retry
      break;
    case 'CAMERA_ACCESS_DENIED':
      // Direct user to device Settings → Privacy → Camera
      break;
    case 'USER_CANCELLED':
      // User dismissed the SDK UI — show alternative login
      break;
    case 'NETWORK_ERROR':
      // No connectivity
      break;
    case 'NO_ACTIVITY':
    case 'NO_PRESENTER':
      // Bridge setup error — check bridge module is registered correctly
      console.error('Entry bridge error:', message);
      break;
    default:
      console.error('Entry error:', code, message);
  }
}
```

---

## Expo-specific notes

Expo Go **does not support native modules**. You must use a dev build.

**With Expo bare workflow** — follow all steps above as written.

**With Expo managed workflow** — generate native projects first:

```bash
npx expo prebuild
```

This creates `ios/` and `android/` directories. Then follow steps 1–4 above in those directories.

To run:
```bash
npx expo run:ios      # builds and installs on connected device
npx expo run:android
```

Or with EAS:
```bash
eas build --platform ios --profile development
eas build --platform android --profile development
```

**Expo `app.json` permissions** (alternative to editing Info.plist / AndroidManifest directly):

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "Entry needs camera access for identity verification."
      }
    },
    "android": {
      "permissions": ["android.permission.CAMERA"]
    }
  }
}
```

---

## Mistakes AI coding assistants commonly make

- **Using `EntrySDK.shared` (iOS)** — the iOS class is `EntrySDKClient.shared`, not `EntrySDK.shared`
- **Calling `EntrySDK.configure()` (Android)** — the correct method is `EntrySDK.initialize()`; `configure()` does not exist
- **Calling `EntrySDK.identifyUser()` directly (Android)** — must go through `EntrySDK.getInstance().identifyUser()`
- **Using `user.fullName`** — the field does not exist; use `user.firstName` and `user.lastName` separately
- **Omitting the `.m` macro file (iOS)** — Swift-to-RCT bridging requires both the `.swift` and the `.m` file; without the `.m` the module is invisible to JS
- **Using `currentActivity` without casting to `AppCompatActivity`** — the Entry Android SDK requires `AppCompatActivity`, not plain `Activity`
- **Testing on Expo Go** — native modules are not available in Expo Go; if `NativeModules.EntryBridgeModule` is undefined, you are running in Expo Go
- **Testing on simulator / emulator** — liveness requires a physical device with a front camera
