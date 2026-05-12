---
layout: default
title: iOS Instructions File
nav_order: 2
parent: AI Integration Context
---

# Entry iOS SDK — AI Integration Context

## What is Entry?

Entry is a **biometric identity verification SDK**. It is a **UI component** — not a raw API. You call `identifyUser()`, the SDK presents its own liveness and face-matching UI, and it returns a user object or throws an error. Do not call the Entry API directly.

## Key facts

- Distributed as a **Swift Package Manager binary** via a private GitHub repository
- SDK is a shared singleton — configure once via `EntrySDKClient.shared.configure()`
- Async/await API — all public methods are `async throws`
- Errors conform to a typed protocol with an error code — not generic `Error`
- Requires a **physical device** with a front camera — the iOS simulator is not supported
- Requires `NSCameraUsageDescription` in `Info.plist`

## Before you can use Entry

Two prerequisites — done by the app owner, not the integrating developer:
1. The app's **bundle identifier** (e.g. `com.yourcompany.yourapp`) must be registered with the Entry team
2. Each developer's **GitHub account** must be invited to the `synapser-sdk-distribution` org

Contact support@synapser.com for both. Accept the GitHub org invitation before trying to add the package in Xcode.

## Environment requirements

| Requirement | Minimum |
|---|---|
| OS | **macOS only** — Xcode is macOS-only, iOS development is not possible on Windows or Linux |
| Xcode | 14+ |
| Swift | 5.7+ |
| iOS deployment target | 14.0+ |
| Device | Physical iPhone with front camera — the iOS Simulator is not supported |

## Adding the SDK (Swift Package Manager)

1. In Xcode: **File → Add Package Dependencies…**
2. Enter the package URL provided by the Entry team
3. Dependency rule: **Up to Next Major Version**
4. Add the `EntrySDK` library to your application target

You must be signed into GitHub in **Xcode → Settings → Accounts** with your invited account.

## Required — Info.plist permission

```xml
<key>NSCameraUsageDescription</key>
<string>Entry needs camera access for identity verification.</string>
```

The App Store will reject the app without this key. Add it before first TestFlight build.

## Configuration (do this once, at app startup)

Call `configure` once in your `AppDelegate` or app entry point:

```swift
import EntrySDK

EntrySDKClient.shared.configure(
    appName: "your-app-name",  // MUST match the name registered with the Entry team
    environment: .test          // .test for development, .live for production
)
```

| Environment | Use for                        |
| ----------- | ------------------------------ |
| `.test`     | Development and integration QA |
| `.live`     | Production                     |

Always use `.test` during development. Switch to `.live` only in production builds.

## Identifying a user (with registration fallback)

Standard flow. If the user is not recognised, they are registered automatically.

```swift
do {
    let user = try await EntrySDKClient.shared.identifyUser(
        registerIfNotFound: true,
        presenter: self   // the UIViewController presenting the SDK UI
    )
    // user.entryUserId, user.firstName, user.lastName, etc.
    print("Identified:", user.entryUserId)
} catch {
    handleEntryError(error)
}
```

## Identifying a user (no registration)

Throws if the user is not already registered.

```swift
let user = try await EntrySDKClient.shared.identifyUser(
    registerIfNotFound: false,
    presenter: self
)
```

## Error handling

Catch errors and check the error code to respond appropriately.

```swift
func handleEntryError(_ error: Error) {
    guard let sdkError = error as? EntrySDKError else {
        // Non-SDK error
        showAlert("An unexpected error occurred.")
        return
    }
    switch sdkError.code {
    case .userNotFound:
        // User is not registered — prompt them to register
        break
    case .livenessCheckFailed:
        // Liveness rejected — show lighting/positioning advice and offer retry
        break
    case .cameraAccessDenied:
        // User denied camera — direct to Settings → Privacy → Camera
        break
    case .userCancelled:
        // User closed the UI — show alternative login options
        break
    case .networkError:
        // No connectivity
        break
    case .invalidAppName:
        // App name mismatch — check configure() call
        break
    default:
        showAlert(sdkError.localizedDescription)
    }
}
```

## Security

- **Never commit the GitHub PAT** — store it in Xcode's credential manager or your CI secrets. A PAT in a committed `.netrc` or `~/.netrc` file is a critical credential leak.
- **Never log the user object** — `identifyUser()` returns PII (name, ID). Do not pass it to `print()`, Crashlytics, or Sentry without stripping sensitive fields.
- **Store any session tokens in the Keychain** — not in `UserDefaults`. `UserDefaults` is not encrypted and can be read by other processes on a jailbroken device.
- **Use the `.test` environment during development** — never use `.live` in dev builds. This prevents polluting production face data.

## Common mistakes — do not do these

- **Do not call the Entry API directly** — `identifyUser()` is the only integration point needed
- **Do not skip `NSCameraUsageDescription`** — the app will crash at runtime without it
- **Do not use `.live` during development** — always use `.test` to avoid affecting production data
- **Do not test on the simulator** — liveness requires a physical device with a front camera
- **Do not forget to accept the GitHub org invitation** — Xcode cannot resolve the package until you accept it
- **Do not configure the SDK multiple times** — call `configure()` once at app startup, not before each `identifyUser()` call- **Do not use `EntrySDK.shared`** — the correct class is `EntrySDKClient.shared`

## Upgrading the SDK

With the recommended `Up to Next Major` rule, upgrade in Xcode: **File → Packages → Resolve Package Versions**. Then test the identify and registration flows before shipping.

## Links

- Full integration guide: https://synapser-limited.github.io/entry-sdk-docs/integration/ios/
- Client onboarding: https://synapser-limited.github.io/entry-sdk-docs/getting-started/client-onboarding/
- Platform requirements: https://synapser-limited.github.io/entry-sdk-docs/getting-started/requirements/
