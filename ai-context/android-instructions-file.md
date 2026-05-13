---
layout: default
title: Android Instructions File
nav_order: 3
parent: AI Integration Context
---

# Entry Android SDK — AI Integration Context

## What is Entry?

Entry is a **biometric identity verification SDK**. It is a **UI component** — not a raw API. You call `EntrySDK.identifyUser()`, the SDK handles liveness detection and face matching internally, and it returns a user object or throws an exception. Do not call the Entry API directly.

## Key facts

- Package: `com.synapser:entry-sdk` (private, Maven via GitHub Packages)
- SDK is a singleton — initialise once via `EntrySDK.initialize()`, then access via `EntrySDK.getInstance()`
- Coroutine-based API — `identifyUser()` is a `suspend fun` returning `Result<EntryUser>`
- Errors are `EntrySDKError` instances with a `.code` property
- Requires a **physical device** with a front camera — the emulator is not supported
- Requires `CAMERA` permission in `AndroidManifest.xml` (the SDK requests it at runtime)

## Before you can use Entry

Two prerequisites — done by the app owner, not the integrating developer:
1. The app's **package name** (e.g. `com.yourcompany.yourapp`) must be registered with the Entry team
2. Each developer's **GitHub account** must be invited to the `synapser-sdk-distribution` org

Contact support@synapser.com for both. Accept the GitHub org invitation before syncing Gradle.

## Environment requirements

| Requirement    | Minimum |
| -----------    | ------- |
| OS             | macOS, Windows, or Linux |
| Android Studio | Hedgehog (2023.1) or later recommended |
| JDK            | 11+ (Gradle 7+ requires it) |
| Kotlin         | 1.7+ |
| Gradle         | 7.0+ |
| minSdk         | 21 (Android 5.0) |
| Device         | Physical Android device with front camera — the emulator is not supported |

## Adding the SDK (Gradle / Maven)

### 1. Store credentials (never in version control)

Add to `~/.gradle/gradle.properties`:
```properties
github.username=YOUR_GITHUB_USERNAME
github.token=YOUR_GITHUB_PAT
```

Your PAT needs `read:packages` scope.

### 2. Configure the Maven repository

In project-level `settings.gradle.kts`:
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

### 3. Add the dependency

In app-level `build.gradle.kts`:
```kotlin
dependencies {
    implementation("com.synapser:entry-sdk:<version>")
}
```

Version number is provided by the Entry team.

## Required — AndroidManifest.xml permission

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="true" />
```

The SDK requests the runtime permission automatically — you do not need to call `requestPermissions()` yourself.

## Configuration (do this once, at app startup)

Call `configure` once in your `Application` class:

```kotlin
import com.synapser.entry.EntrySDK
import com.synapser.entry.EntryEnvironment

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        EntrySDK.initialize(
            context = this,
            appName = "your-app-name",        // MUST match the name registered with the Entry team
            environment = EntryEnvironment.TEST // TEST for development, LIVE for production
        )
    }
}
```

| Environment             | Use for                        |
| ----------------------- | ------------------------------ |
| `EntryEnvironment.TEST` | Development and integration QA |
| `EntryEnvironment.LIVE` | Production                     |

Always use `TEST` during development. Switch to `LIVE` only in production builds.

Register `MyApplication` in `AndroidManifest.xml`:
```xml
<application android:name=".MyApplication" ...>
```

## Identifying a user (with registration fallback)

Standard flow. If the user's face is not recognised, they are registered automatically.

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
                    handleEntryError(error as? EntrySDKError ?: return@onFailure)
                }
        }
    }
}
```

## Identifying a user (no registration)

Throws if the user is not already registered.

```kotlin
val result = EntrySDK.getInstance().identifyUser(
    registerIfNotFound = false,
    activity = this
)
```

## Error handling

```kotlin
import com.synapser.entry.models.EntrySDKError
import com.synapser.entry.models.EntrySDKErrorCode

fun handleEntryError(e: EntrySDKError) {
    when (e.code) {
        EntrySDKErrorCode.USER_NOT_FOUND ->
            // User passed liveness but is not registered — prompt to register
            promptRegistration()
        EntrySDKErrorCode.LIVENESS_CHECK_FAILED ->
            // Liveness rejected — show tips and allow retry
            showMessage("Please try again in better lighting.")
        EntrySDKErrorCode.CAMERA_ACCESS_DENIED ->
            // Direct user to App Settings → Permissions → Camera
            showCameraPermissionInstructions()
        EntrySDKErrorCode.USER_CANCELLED ->
            // User dismissed the UI — show alternative login
            showLoginOptions()
        EntrySDKErrorCode.NETWORK_ERROR ->
            showOfflineMessage()
        EntrySDKErrorCode.INVALID_APP_NAME ->
            // App name mismatch — check initialize() call
            Log.e("Entry", "App name not registered: ${e.message}")
        else ->
            showError(e.userMessage ?: e.message ?: "An error occurred")
    }
}
```

## Security

- **Never commit GitHub credentials** — keep `github.username` and `github.token` in `~/.gradle/gradle.properties`, which is outside the project directory. Never add them to `gradle.properties` inside the project (that file gets committed).
- **Never log the user object** — `identifyUser()` returns PII (name, ID). Do not pass it to `Log.d()`, Firebase Crashlytics, or Sentry without stripping sensitive fields.
- **Store any session tokens in `EncryptedSharedPreferences`** — not plain `SharedPreferences`. Plain preferences are stored unencrypted on the device filesystem.
- **Use the `TEST` environment during development** — never use `LIVE` in dev builds. This prevents polluting production face data.

## Common mistakes — do not do these

- **Do not call the Entry API directly** — `identifyUser()` is the only integration point needed
- **Do not commit GitHub credentials** — keep credentials in `~/.gradle/gradle.properties`, not in the project
- **Do not use `LIVE` during development** — always use `TEST` to avoid affecting production data
- **Do not test on the emulator** — liveness requires a physical device with a front camera
- **Do not forget to accept the GitHub org invitation** — Gradle cannot resolve the package until you accept it
- **Do not use `EntrySDK.configure()`** — the method is `EntrySDK.initialize()`. `configure()` does not exist.
- **Do not call `EntrySDK.identifyUser()` directly** — always go through `EntrySDK.getInstance().identifyUser()`
- **Do not configure the SDK multiple times** — call `initialize()` once in `Application.onCreate()`, not in Activity
- **Do not forget to declare the `Application` class** in `AndroidManifest.xml`

## Upgrading the SDK

Update the version string in `build.gradle.kts` and sync Gradle. Test the identify and registration flows after upgrading.

## Links

- Full integration guide: https://synapser-limited.github.io/entry-sdk-docs/integration/android/
- Client onboarding: https://synapser-limited.github.io/entry-sdk-docs/getting-started/client-onboarding/
- Platform requirements: https://synapser-limited.github.io/entry-sdk-docs/getting-started/requirements/
