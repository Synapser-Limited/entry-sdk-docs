---
layout: default
title: Android SDK
nav_order: 8
parent: Integration Guides
---

# Android SDK Integration Guide

This guide covers integrating the Entry Android SDK into an Android application for biometric identity verification.

## Requirements

- Android API 21+ (Android 5.0 Lollipop)
- Kotlin 1.7+, Gradle 7.0+
- Physical device with front camera (emulator not supported for liveness)
- Your application registered with the Entry team — see [Client Onboarding](../getting-started/client-onboarding.md)

## 1) Getting access

The Entry Android SDK is distributed via Maven on GitHub Packages. To get access:

1. Provide the Entry team with:
   - The **GitHub username** of each developer who needs access
   - Your application's **package name** (e.g. `com.yourcompany.yourapp`)
2. The Entry team will add your GitHub accounts to the `synapser-sdk-distribution` org and provide the SDK version to use.

> Accept the GitHub organisation invitation before proceeding.

## 2) Adding the SDK to your project

Configure Gradle to authenticate with GitHub Packages. In your project-level `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/Synapser-Limited/entry-web-sdk")
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

Add your GitHub credentials to `~/.gradle/gradle.properties` (not in version control):

```properties
github.username=YOUR_GITHUB_USERNAME
github.token=YOUR_GITHUB_PAT
```

Add the dependency to your app-level `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.synapser:entry-sdk:<version>")
}
```

## 3) Required permissions

Add the camera permission to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="true" />
```

The SDK handles the runtime permission request. You do not need to request it manually.

## 4) SDK setup

### Configure on application start

Call `initialize` once, typically in your `Application` class:

```kotlin
import com.synapser.entry.EntrySDK
import com.synapser.entry.EntryEnvironment

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        EntrySDK.initialize(
            context = this,
            appName = "your-app-name",   // provided by the Entry team
            environment = EntryEnvironment.LIVE  // LIVE or TEST
        )
    }
}
```

| Environment             | Purpose                             |
| ----------------------- | ----------------------------------- |
| `EntryEnvironment.TEST` | Integration testing and development |
| `EntryEnvironment.LIVE` | Production                          |

> Use `TEST` during development. Switch to `LIVE` for production builds.

### Identify user (with registration fallback)

If the user's face is not recognised, they are automatically registered:

```kotlin
import com.synapser.entry.EntrySDK
import com.synapser.entry.models.EntrySDKError

class MainActivity : AppCompatActivity() {

    private fun authenticate() {
        lifecycleScope.launch {
            val result = EntrySDK.getInstance().identifyUser(
                registerIfNotFound = true,
                activity = this@MainActivity
            )
            result
                .onSuccess { user ->
                    // user.entryUserId, user.firstName, user.lastName, etc.
                    onUserIdentified(user)
                }
                .onFailure { error ->
                    handleError(error as? EntrySDKError ?: return@onFailure)
                }
        }
    }
}
```

### Identify only (no registration)

Returns a failure result if the user is not already registered:

```kotlin
val result = EntrySDK.getInstance().identifyUser(
    registerIfNotFound = false,
    activity = this
)
```

## 5) Error handling

```kotlin
import com.synapser.entry.models.EntrySDKError
import com.synapser.entry.models.EntrySDKErrorCode

fun handleError(e: EntrySDKError) {
    when (e.code) {
        EntrySDKErrorCode.USER_NOT_FOUND ->
            promptRegistration()
        EntrySDKErrorCode.LIVENESS_CHECK_FAILED ->
            showMessage("Liveness check failed. Please try again.")
        EntrySDKErrorCode.CAMERA_ACCESS_DENIED ->
            showCameraInstructions()
        EntrySDKErrorCode.USER_CANCELLED ->
            showLoginOptions()
        EntrySDKErrorCode.NETWORK_ERROR ->
            showOfflineMessage()
        else ->
            showError(e.userMessage ?: e.message ?: "An error occurred")
    }
}
```

## 6) Upgrading the SDK

Update the version in your `build.gradle.kts` and sync the project:

```kotlin
implementation("com.synapser:entry-sdk:<new-version>")
```

Test the identify and registration flows after upgrading.

## 7) Troubleshooting

### Package fails to resolve

- Confirm your `gradle.properties` contains valid GitHub credentials.
- Your PAT must have `read:packages` scope.
- Your GitHub account must have accepted the `synapser-sdk-distribution` org invitation.

### Camera permission denied

The SDK requests the `CAMERA` permission at runtime. If the user has permanently denied it, direct them to App Settings → Permissions.

### Liveness fails on device

- Confirm the device has a working front camera.
- Ensure adequate lighting.
- Emulators are not supported — use a physical device.

### Build fails after upgrade

Clean and rebuild: **Build → Clean Project**, then **Build → Rebuild Project**.

## 8) Verification checklist

| Step | Action                             | Expected result                                       |
| ---- | ---------------------------------- | ----------------------------------------------------- |
| 1    | Sync Gradle                        | No dependency resolution errors                       |
| 2    | Build the project                  | No compile errors                                     |
| 3    | Run on a physical device           | App launches, SDK initialises                         |
| 4    | Trigger **Identify with Register** | Liveness UI appears; user is identified or registered |
| 5    | Trigger **Identify Only**          | Liveness UI appears; registered user is identified    |

## 9) Support

Contact [support@synapser.com](mailto:support@synapser.com) with:

- Your app's package name and environment
- Android version and device model
- The full error message or error code
- Steps to reproduce
