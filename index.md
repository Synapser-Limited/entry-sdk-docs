---
layout: default
title: Home
nav_order: 1
description: "Entry SDK - Facial biometric authentication for Web, iOS, and Android. Passwordless login with liveness detection, powered by AWS Rekognition."
keywords: facial recognition, biometric authentication, passwordless login, liveness detection, face verification, AWS Rekognition, iOS SDK, Android SDK, TypeScript SDK, React authentication, web biometrics
permalink: /
---

# Entry SDK Documentation

{: .fs-9 }

Passwordless facial biometric authentication for Web, iOS, and Android.
{: .fs-6 .fw-300 }

{: .label .label-green }

[Get Started](./getting-started/overview){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Client Onboarding](./getting-started/client-onboarding){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is Entry?

Entry enables **passwordless authentication** using facial biometrics with active liveness detection. Built on AWS Rekognition, it provides:

- **Facial Recognition** — Identify and verify users by their face
- **Liveness Detection** — Prevent spoofing with active liveness checks
- **User Registration** — Seamless onboarding with biometric enrolment
- **Multi-Platform** — Native SDKs for Web (TypeScript/React), iOS (Swift), and Android (Kotlin)
- **Enterprise Security** — AWS-powered backend, encrypted data transmission

{: .warning }
> **Physical camera required** — Liveness detection requires a real camera. Simulators and emulators are not supported.

---

## Platform Guides

| Platform                   | Integration Guide                                                      |
| :------------------------- | :--------------------------------------------------------------------- |
| 🌐 Web (TypeScript/React)  | [Web Integration](./ai-context/web-instructions-file/)                 |
| 🍎 iOS (Swift)             | [iOS Integration](./-context/ios-instructions-file/)                   |
| 🤖 Android (Kotlin)        | [Android Integration](./-context/android-instructions-file/)           |
| 📱 React Native            | [React Native Integration](./-context/react-native-instructions-file/) |

## AI Prompt Library

| Documentation                                                                                                                                                                                            | Description                          |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------- |
| � [Prompt Library](https://synapser-limited.github.io/entry-sdk-docs/ai-context/prompt-library/)                                                                           | AI prompt library for Entry SDK      |
| ✅ [Integration Checklist](https://synapser-limited.github.io/entry-sdk-docs/ai-context/integration-checklist/)                                                     | Integration checklist for Entry SDK  |

## Downloadable AI Context Files

| Documentation                                                                                                                                                                                            | Description                          |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------- |
| 🤖 [Web AI Instructions](https://raw.githubusercontent.com/Synapser-Limited/entry-sdk-docs/main/ai-context/web-instructions-file.md){: download="web-instructions-file.md" }                             | AI context file for Web SDK          |
| 🍎 [iOS AI Instructions](https://raw.githubusercontent.com/Synapser-Limited/entry-sdk-docs/main/ai-context/ios-instructions-file.md){: download="ios-instructions-file.md" }                             | AI context file for iOS SDK          |
| 🤖 [Android AI Instructions](https://raw.githubusercontent.com/Synapser-Limited/entry-sdk-docs/main/ai-context/android-instructions-file.md){: download="android-instructions-file.md" }                 | AI context file for Android SDK      |
| 📱 [React Native AI Instructions](https://raw.githubusercontent.com/Synapser-Limited/entry-sdk-docs/main/ai-context/react-native-instructions-file.md){: download="react-native-instructions-file.md" }  | AI context file for React Native SDK |
| � [Prompt Library](https://synapser-limited.github.io/entry-sdk-docs/ai-context/prompt-library/){: download="prompt-library" }                                                                           | AI prompt library for Entry SDK      |
| ✅ [Integration Checklist](https://synapser-limited.github.io/entry-sdk-docs/ai-context/integration-checklist/){: download="integration-checklist" }                                                     | Integration checklist for Entry SDK  |

## Other Resources

| Documentation                                                                                                 | Description                       |
| :------------------------------------------------------------------------------------------------------------ | :-------------------------------- |
| 🚀 [Client Onboarding](https://synapser-limited.github.io/entry-sdk-docs/getting-started/client-onboarding/)  | Step-by-step onboarding guide     |
| � [Changelog](./changelog)                                                                                    | Version history and release notes |
| 📄 [License](./license)                                                                                       | Software license terms            |

---

## Getting Started (Web)

```typescript
import { EntrySDK, EntryApiEnvironment, EntrySDKError } from '@synapser-sdk-distribution/entry-web-sdk';

// Initialise SDK once (app name provided by Synapser on registration)
const entrySDK = EntrySDK.getInstance(
  'your-app-name',
  EntryApiEnvironment.Live
);

// Identify user — liveness check runs inside the container element
async function authenticateUser() {
  try {
    const user = await entrySDK.identifyUser(
      true,  // Register if not found
      document.getElementById('auth-container')!
    );
    console.log('Authenticated:', user.entryUserId);
  } catch (error) {
    if (error instanceof EntrySDKError) {
      console.error(`Error ${error.code}: ${error.message}`);
    }
  }
}
```

For complete integration details, see the platform guides above.

---

## Support

For support enquiries, contact [support@synapser.com](mailto:support@synapser.com).
