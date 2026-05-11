---
layout: default
title: Platform Overview
nav_order: 1
parent: Getting Started
---

# Platform Overview

Entry is a biometric identity verification platform. It enables applications to verify and identify users through face liveness detection, without passwords or PINs.

## How it works

```
Customer App
  │
  ├── Entry SDK (Web / iOS / Android)
  │     │
  │     ├── Liveness check — user looks at their camera; AWS Rekognition
  │     │   confirms the user is physically present (not a photo or replay)
  │     │
  │     └── Identification — the verified face is matched against registered
  │         users; the SDK returns the matched user record
  │
  └── Entry API — orchestrates registration, identification, and face search
        │
        ├── AWS Rekognition — face indexing and search
        ├── Amazon Aurora MySQL — user records
        └── Amazon S3 — secure file storage
```

## Key flows

### Identify with registration fallback

The most common flow. If the user's face is recognised, their record is returned immediately. If not, the SDK walks them through registration.

```
User → Liveness check → Face search
  Found:      return user record
  Not found:  register user → return new user record
```

### Identify only

The user must already be registered. Returns an error if no match is found.

```
User → Liveness check → Face search
  Found:      return user record
  Not found:  return USER_NOT_FOUND error
```

## Supported platforms

| Platform | SDK                                              | Minimum OS                                    |
| -------- | ------------------------------------------------ | --------------------------------------------- |
| Web      | `@synapser-sdk-distribution/entry-web-sdk` (npm) | Chrome 80+, Firefox 75+, Safari 13+, Edge 80+ |
| iOS      | `EntrySDK` (Swift Package Manager)               | iOS 14.0+                                     |
| Android  | `com.synapser:entry-sdk` (Maven)                 | Android API 21+                               |

All platforms require a **physical camera**. The simulator and emulator are not supported for liveness detection.

## Environments

| Environment | Purpose            | API base                              |
| ----------- | ------------------ | ------------------------------------- |
| Test        | Integration and QA | `entry-lite-api.test.uk.entrymfa.com` |
| Live        | Production         | `entry-lite-api.live.uk.entrymfa.com` |

## Before you start

1. Contact the Entry team to register your application — see [Client Onboarding](client-onboarding.md).
2. Check [Requirements](requirements.md) for your platform.
3. Follow the integration guide for your SDK: [iOS](../integration/ios.md) · [Android](../integration/android.md) · [Web](../integration/web.md).
