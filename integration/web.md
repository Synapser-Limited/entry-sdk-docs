---
layout: default
title: Web Integration Guide
nav_order: 1
---

# Web SDK Integration Guide

This guide covers integrating the Entry Web SDK into a web application for biometric identity verification.

## Requirements

- Node.js 18+, npm 7+
- HTTPS-enabled application (required for camera access; `localhost` is exempt during development)
- Chrome 80+, Firefox 75+, Safari 13+, or Edge 80+
- Your application registered with the Entry team — see [Client Onboarding](../getting-started/client-onboarding.md)

## 1) Installation

Configure GitHub Packages authentication once per machine:

```ini
@synapser-sdk-distribution:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```

Save that in `~/.npmrc`, then set `NODE_AUTH_TOKEN` in your shell or secret manager before running `npm install`.

> Do not store your GitHub PAT directly in `~/.npmrc`, commit it to source control, or paste it into shell commands that may be saved in shell history.

Then install:

```bash
npm install @synapser-sdk-distribution/entry-web-sdk
```

## 2) HTML structure

The SDK renders its liveness UI (camera view) inside a container element. The element must be able to expand to cover the viewport.

```html
<body>
  <!-- your app content -->

  <!-- Entry SDK overlay container -->
  <div id="entry-container"></div>
</body>
```

## 3) SDK setup

Initialise the SDK once, typically in your app's entry point:

```typescript
import { EntrySDK, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

const sdk = EntrySDK.getInstance(
  'your-app-name',        // provided by the Entry team
  EntryApiEnvironment.Live // .Test or .Live
);
```

| Environment                | Purpose                             |
| -------------------------- | ----------------------------------- |
| `EntryApiEnvironment.Test` | Integration testing and development |
| `EntryApiEnvironment.Live` | Production                          |

> Use `.Test` during development. Switch to `.Live` for production builds.

## 4) Identify user (with registration fallback)

If the user's face is not recognised, they are automatically registered:

```typescript
import { EntrySDK, EntrySDKError, EntrySDKErrorCode, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

async function authenticate() {
  const sdk = EntrySDK.getInstance('your-app-name', EntryApiEnvironment.Live);
  const container = document.getElementById('entry-container');

  try {
    const user = await sdk.identifyUser(
      true,      // registerIfNotFound
      container
    );
    console.log('User identified:', obfuscatePII(user));
  } catch (error) {
    if (error instanceof EntrySDKError) {
      handleError(error);
    }
  }
}
```

## 5) Identify only (no registration)

Returns an error if the user is not already registered:

```typescript
const user = await sdk.identifyUser(
  false,     // registerIfNotFound — will throw USER_NOT_FOUND if not registered
  container
);
```

## 6) Error handling

```typescript
import { EntrySDKError, EntrySDKErrorCode } from '@synapser-sdk-distribution/entry-web-sdk';

function handleError(error: EntrySDKError) {
  switch (error.code) {
    case EntrySDKErrorCode.USER_NOT_FOUND:
      // User completed liveness but is not registered
      promptRegistration();
      break;
    case EntrySDKErrorCode.LIVENESS_CHECK_FAILED:
      // Face liveness was not confirmed
      if (error.isRetryable()) showRetry('Please try again in better lighting.');
      break;
    case EntrySDKErrorCode.CAMERA_ACCESS_DENIED:
      // User denied camera permission
      showCameraInstructions();
      break;
    case EntrySDKErrorCode.USER_CANCELLED:
      // User closed the liveness UI
      showLoginOptions();
      break;
    case EntrySDKErrorCode.NETWORK_ERROR:
      showOfflineMessage();
      break;
    default:
      showError(error.message);
  }
}
```

## 7) Troubleshooting

### Camera not accessible

- Ensure the page is served over **HTTPS** (not HTTP). Browsers only allow camera access on secure origins.
- Check the browser console for permission errors.
- If the user denied camera access, they must reset it in their browser settings.

### SDK fails to initialise

- Verify your app name matches the name registered with the Entry team.
- Confirm you are pointing to the correct environment (`.Test` vs `.Live`).

### Liveness fails consistently

- Ensure adequate lighting — the camera needs to see the user's face clearly.
- Test on a physical device or machine with a working camera.
- Check that the entry container element is visible and not obscured.

### Package install fails

- Confirm `~/.npmrc` is configured correctly with your GitHub PAT.
- Your PAT must have `read:packages` scope.
- Your GitHub account must be a member of the `synapser-sdk-distribution` org.

## 8) Verification checklist

| Step | Action                             | Expected result                                       |
| ---- | ---------------------------------- | ----------------------------------------------------- |
| 1    | Install package                    | `npm install` completes without errors                |
| 2    | Initialise SDK                     | No error on `getInstance()`                           |
| 3    | Open page over HTTPS               | Camera permission prompt appears                      |
| 4    | Trigger **Identify with Register** | Liveness UI appears; user is identified or registered |
| 5    | Trigger **Identify Only**          | Liveness UI appears; registered user is identified    |

## 9) Support

Contact [support@synapser.com](mailto:support@synapser.com) with:

- Your app name and environment
- Browser and OS version
- The full error message or error code
- A description of the steps to reproduce
