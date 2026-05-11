---
layout: default
title: iOS SDK
nav_order: 7
parent: Integration Guides
---

# iOS SDK Integration Guide

This guide covers everything you need to integrate the Entry iOS SDK into your application for biometric identity verification.

## Requirements

- iOS 14.0+
- Xcode 14+
- Swift 5.7+
- Camera-capable device (simulator not supported for liveness)

## 1) Getting access

The Entry iOS SDK is distributed as a pre-built binary via a private Swift Package Manager repository. To get access:

1. Provide the Entry team with:
   - Your **GitHub username** (each developer who needs access)
   - Your application's **bundle identifier** (e.g. `com.yourcompany.yourapp`)
2. The Entry team will grant your GitHub account read access to the SDK package repository and provide you with the **package URL**.

> You will receive an invitation to the private repository on GitHub. Accept it before proceeding.

## 2) Adding the SDK to your project

1. Open your iOS project in Xcode.
2. Go to **File → Add Package Dependencies…**
3. Enter the package URL provided by the Entry team.
4. Set the dependency rule to **Up to Next Major Version**.
5. Add the `EntrySDK` library to your application target.

## 3) Required permissions

Add the following to your `Info.plist`:

| Key                        | Required | Purpose                                  |
| -------------------------- | -------- | ---------------------------------------- |
| `NSCameraUsageDescription` | Yes      | Liveness detection uses the front camera |

Example:

```xml
<key>NSCameraUsageDescription</key>
<string>Entry needs camera access for identity verification.</string>
```

## 4) SDK setup

### Configure on app startup

Call `configure` once, typically in your `AppDelegate` or app entry point:

```swift
import EntrySDK

EntrySDK.shared.configure(
    appName: "<your-app-name>",   // provided by the Entry team
    environment: .test             // .test or .live
)
```

| Environment | Purpose                             |
| ----------- | ----------------------------------- |
| `.test`     | Integration testing and development |
| `.live`     | Production                          |

> Start with `.test` during development. Switch to `.live` for production builds.

### Identify user (with registration fallback)

If the user is not recognized, the SDK will register them automatically:

```swift
let user = try await EntrySDK.shared.identifyUser(
    registerIfNotFound: true,
    presenter: viewController
)
```

### Identify only (no registration)

Returns an error if the user is not already registered:

```swift
let user = try await EntrySDK.shared.identifyUser(
    registerIfNotFound: false,
    presenter: viewController
)
```

The `presenter` parameter is the `UIViewController` that the SDK will present its liveness UI from.

## 5) Upgrading the SDK

With the recommended `Up to Next Major` dependency rule, upgrading is straightforward:

1. In Xcode: **File → Packages → Resolve Package Versions**
2. Build and test the identify and registration flows.
3. If you encounter a regression, temporarily pin to the previous version with `exact:` and report the issue to the Entry team.

## 6) Troubleshooting

### Package fails to resolve

- **Accept the GitHub invitation.** You must accept the repository invite before Xcode can access the package.
- **Authenticate Xcode with GitHub.** Go to Xcode → Settings → Accounts and confirm your GitHub account is signed in.
- **Corporate network/proxy.** Ensure your network allows access to `github.com`.

### Binary download fails after resolution

- Check your network connection — the binary artifact is downloaded separately from the package manifest.
- Try resetting Xcode's package cache: **File → Packages → Reset Package Caches**, then resolve again.

### Build succeeds but liveness fails at runtime

- Ensure you are running on a **physical device** — the camera is required for liveness and is not available on the simulator.
- Verify `NSCameraUsageDescription` is set in `Info.plist`.
- Confirm the correct `environment` is set in `configure()`.

### General cache reset

If Xcode behaves unexpectedly with package resolution:

1. **File → Packages → Reset Package Caches**
2. **File → Packages → Resolve Package Versions**
3. Clean build folder: **Product → Clean Build Folder** (⇧⌘K)

## 7) Verification checklist

Use this after initial setup or after upgrading to confirm everything works end-to-end.

| Step | Action                                                 | Expected result                                       |
| ---- | ------------------------------------------------------ | ----------------------------------------------------- |
| 1    | Add package in Xcode using the URL from the Entry team | Package resolves without errors                       |
| 2    | Build the project                                      | No download or checksum errors                        |
| 3    | Run on a physical device                               | App launches, SDK initializes                         |
| 4    | Trigger **Identify with Register**                     | Liveness UI appears, user is identified or registered |
| 5    | Trigger **Identify Only**                              | Liveness UI appears, user is identified               |

## 8) Support

If you encounter issues not covered here, contact the Entry team with:

- Your GitHub username
- Your app's bundle identifier
- Xcode version
- iOS version of test device
- The error message or screenshot

---

> **Dual-maintained file:** This document is also present at `entry-ios-sdk-binary/IOS_INTEGRATION_GUIDE.md` for SPM consumers who read integration guides directly from the binary distribution repo. Any change to this file must be mirrored there. See [CONTRIBUTING.md](../../CONTRIBUTING.md).
