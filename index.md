---
layout: default
title: Home
nav_order: 1
description: "Entry Web SDK - Facial biometric authentication for web apps. Passwordless login with liveness detection, powered by AWS Rekognition. TypeScript/React SDK."
keywords: facial recognition, biometric authentication, passwordless login, liveness detection, face verification, AWS Rekognition, TypeScript SDK, React authentication, web biometrics
permalink: /
---

# Entry Web SDK Documentation

{: .fs-9 }

Passwordless facial biometric authentication for web applications.
{: .fs-6 .fw-300 }

{: .label .label-green }

[Get Started](./integration){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Quick Reference](./quick-reference){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is Entry Web SDK?

Entry Web SDK enables **passwordless authentication** using facial biometrics with active liveness detection. Built on AWS Rekognition, it provides:

- **Facial Recognition** - Identify and verify users by their face
- **Liveness Detection** - Prevent spoofing with active liveness checks
- **User Registration** - Seamless onboarding with biometric enrollment
- **TypeScript/React** - First-class support for modern web apps
- **Enterprise Security** - AWS-powered backend, encrypted data transmission

{: .warning }
> **HTTPS Required** - The SDK requires HTTPS for camera access. It will not work on `http://` URLs (except `localhost` for development).

## Quick Links

| Documentation                           | Description                                 |
|:----------------------------------------|:--------------------------------------------|
| 🚀 [Integration Guide](./integration)   | Complete setup and integration instructions |
| ⚡ [Quick Reference](./quick-reference)  | Copy-paste code snippets and lookup tables  |
| 🔧 [Troubleshooting](./troubleshooting) | Common issues, FAQs, and debugging tips     |
| 🔒 [Security](./security)               | Security guidelines and best practices      |
| ⚠️ [Error Handling](./error-handling)   | Error codes and handling strategies         |
| 📖 [Use Cases](./use-cases)             | Common integration scenarios                |
| 📋 [Changelog](./changelog)             | Version history and release notes           |
| 📄 [License](./license)                 | Software license terms                      |

---

## Installation

```bash
# Configure GitHub Packages auth (once)
echo "@synapser-limited:registry=https://npm.pkg.github.com" >> ~/.npmrc
echo "//npm.pkg.github.com/:_authToken=YOUR_GITHUB_TOKEN" >> ~/.npmrc

# Install the SDK
npm install @synapser-limited/entry-web-sdk
```

{: .note }
> You need a GitHub Personal Access Token with `read:packages` scope. [Learn how to create one](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

---

## Getting Started

```typescript
import { EntrySDK, EntryApiEnvironment, EntrySDKError } from '@synapser-limited/entry-web-sdk';

// Initialize SDK (provide your app name from Synapser)
const entrySDK = EntrySDK.getInstance(
  'your-app-name-registered-with-synapser',
  EntryApiEnvironment.Live
);

// Identify user with biometric liveness check
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

{: .important }
> **Requirements:**
>
> - HTTPS domain (required for camera access)
> - Container element for the authentication UI
> - Valid app configuration from Synapser

For complete integration details, see the [Integration Guide](./integration).

---

## Support

For support inquiries, please contact [support@synapser.com](mailto:support@synapser.com).
