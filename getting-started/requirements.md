---
layout: default
title: Requirements
nav_order: 2
---

# Requirements

## All platforms

- A **physical camera** on the device. Liveness detection requires the front camera; it cannot run in a browser simulator, iOS simulator, or Android emulator.
- An **Entry application registration** — your app's bundle/package ID must be registered with the Entry team before calls to the API will succeed. See [Client Onboarding](client-onboarding.md).
- **HTTPS** for web applications. Browsers only grant camera access on secure origins. `localhost` is the only exception and only during development.

## Web SDK

| Requirement       | Minimum                    |
| ----------------- | -------------------------- |
| Node.js           | 18+                        |
| npm               | 7+                         |
| Browser — Chrome  | 80+                        |
| Browser — Firefox | 75+                        |
| Browser — Safari  | 13+                        |
| Browser — Edge    | 80+                        |
| Camera resolution | 720p                       |
| Protocol          | HTTPS (camera requirement) |

**GitHub access:** The Web SDK is distributed via GitHub Packages. You need a GitHub Personal Access Token (PAT) with `read:packages` scope. See [Client Onboarding](client-onboarding.md) for setup instructions.

## iOS SDK

| Requirement | Minimum                           |
| ----------- | --------------------------------- |
| iOS         | 14.0+                             |
| Xcode       | 14+                               |
| Swift       | 5.7+                              |
| Device      | Physical device with front camera |

**GitHub access:** The iOS SDK is distributed as a Swift Package Manager binary target via a private GitHub repository. You need a GitHub account added to the `synapser-sdk-distribution` org. See [Client Onboarding](client-onboarding.md).

**Permissions:** Add `NSCameraUsageDescription` to your `Info.plist` before submitting to the App Store.

## Android SDK

| Requirement       | Minimum                           |
| ----------------- | --------------------------------- |
| Android API level | 21 (Android 5.0)                  |
| Kotlin            | 1.7+                              |
| Gradle            | 7.0+                              |
| Device            | Physical device with front camera |

**GitHub access:** The Android SDK is distributed via Maven on GitHub Packages. You need a GitHub PAT with `read:packages` scope. See [Client Onboarding](client-onboarding.md).

**Permissions:** Add `CAMERA` permission to your `AndroidManifest.xml`.
