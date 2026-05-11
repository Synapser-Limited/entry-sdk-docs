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

| Platform                  | Integration Guide                            | Quick Reference                                      |
|:--------------------------|:---------------------------------------------|:-----------------------------------------------------|
| 🌐 Web (TypeScript/React) | [Web Integration](./integration/web)         | [Quick Reference](./integration/web-quick-reference) |
| 🍎 iOS (Swift)            | [iOS Integration](./integration/ios)         | —                                                    |
| 🤖 Android (Kotlin)       | [Android Integration](./integration/android) | —                                                    |

## Additional Web SDK Resources

| Documentation                                           | Description                            |
|:--------------------------------------------------------|:---------------------------------------|
| 🔒 [Security](./integration/web-security)               | Security guidelines and best practices |
| ⚠️ [Error Handling](./integration/web-error-handling)   | Error codes and handling strategies    |
| 📖 [Use Cases](./integration/web-use-cases)             | Common integration scenarios           |
| 🔧 [Troubleshooting](./integration/web-troubleshooting) | Common issues and debugging tips       |

## Other Resources

| Documentation               | Description                       |
|:----------------------------|:----------------------------------|
| 📋 [Changelog](./changelog) | Version history and release notes |
| 📄 [License](./license)     | Software license terms            |

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
