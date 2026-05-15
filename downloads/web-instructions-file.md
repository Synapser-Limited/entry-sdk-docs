---
applyTo: "**"
---

# Entry Web SDK — AI Integration Context

## What is Entry?

Entry is a **biometric identity verification SDK**. It is a **UI component** — not a raw API. You mount it, it handles liveness detection and face matching internally, and it returns a user object or throws an error. Do not call the Entry API directly.

## Key facts

- Package: `@synapser-sdk-distribution/entry-web-sdk` (private, GitHub Packages)
- The SDK renders its own camera/liveness UI inside a container element you provide
- The SDK is a singleton — call `EntrySDK.getInstance()` once and reuse it
- Async/await API — all methods return Promises
- Errors are `EntrySDKError` instances with a `.code` property (not generic `Error`)
- Requires HTTPS (camera access). `localhost` is exempt during development

## Before you can use Entry

Two prerequisites must be in place — these are done by the app owner, not the developer integrating:
1. The application must be **registered** with the Entry team (they need your app name and bundle/package ID)
2. Your GitHub account must be added to the `synapser-sdk-distribution` org to access the private package

Contact support@synapser.com for both.

## Environment requirements

| Requirement | Minimum |
|---|---|
| OS | macOS, Windows, or Linux |
| Node.js | 18+ |
| npm | 7+ |
| Browser | Chrome 80+, Firefox 75+, Safari 13+, Edge 80+ |
| Protocol | HTTPS (camera). `localhost` is exempt during development |

## Installation

### 1. Configure GitHub Packages auth (once per machine)

Add to `~/.npmrc`:
```
@synapser-sdk-distribution:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_PAT_READ_PACKAGES}
```

Set `GITHUB_PAT_READ_PACKAGES` in your shell or CI secrets — never hard-code the PAT.

### 2. Install

```bash
npm install @synapser-sdk-distribution/entry-web-sdk
```

## HTML — container element

The SDK renders inside a container element. It must be able to expand to cover the viewport:

```html
<div id="entry-container"></div>
```

## Initialisation (do this once, at app startup)

```typescript
import { EntrySDK, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

const sdk = EntrySDK.getInstance(
  'your-app-name',           // MUST match the name registered with the Entry team
  EntryApiEnvironment.Test   // Use .Test for development, .Live for production
);
```

`getInstance()` returns the same singleton on every call. Do not construct `EntrySDK` directly.

## Environment values

| Value                       | Use for                        |
| --------------------------- | ------------------------------ |
| `EntryApiEnvironment.Test`  | Development and integration QA |
| `EntryApiEnvironment.Live`  | Production                     |
| `EntryApiEnvironment.Demo`  | Demos and proof-of-concept     |

Always default to `.Test` during development. Switch to `.Live` only in production builds.

## Identifying a user (with registration fallback)

This is the standard flow. If the user's face is not recognised, they are registered automatically.

```typescript
const container = document.getElementById('entry-container') as HTMLElement;

try {
  const user = await sdk.identifyUser(
    true,      // registerIfNotFound — true = register automatically if not found
    container
  );
  // user.entryUserId, user.firstName, user.lastName, etc.
  console.log('Identified user:', user.entryUserId);
} catch (error) {
  if (error instanceof EntrySDKError) {
    handleEntryError(error);
  }
}
```

## Identifying a user (no registration)

Returns `USER_NOT_FOUND` error if the user has not previously registered.

```typescript
const user = await sdk.identifyUser(false, container);
```

## EntryUser shape

```typescript
interface EntryUser {
  entryUserId: string;
  firstName: string;
  lastName: string;
  emailAddress: string;
  mobileNumber: string;
  dateOfBirth: string;         // ISO 8601
  gender: string;              // "M" or "F"
  nationalityCountryCodeIso: string;
  photoIdentityDocumentType: string;
  photoIdentityDocumentNumber: string;
  deviceId: string;
  isRegistrationComplete: boolean;
}
```

## Error handling

All errors are `EntrySDKError` instances. Always check `instanceof EntrySDKError` before reading `.code`.

```typescript
import { EntrySDKError, EntrySDKErrorCode } from '@synapser-sdk-distribution/entry-web-sdk';

function handleEntryError(error: EntrySDKError) {
  switch (error.code) {
    case EntrySDKErrorCode.USER_NOT_FOUND:
      // User passed liveness but is not registered — prompt to register
      break;
    case EntrySDKErrorCode.LIVENESS_CHECK_FAILED:
      // Liveness rejected — retryable. Show lighting/positioning tips.
      if (error.isRetryable()) showRetryPrompt();
      break;
    case EntrySDKErrorCode.CAMERA_ACCESS_DENIED:
      // User denied camera permission — show instructions to enable it
      break;
    case EntrySDKErrorCode.USER_CANCELLED:
      // User closed the liveness UI — show alternative login
      break;
    case EntrySDKErrorCode.NETWORK_ERROR:
      // No connectivity
      break;
    case EntrySDKErrorCode.INVALID_APP_NAME:
      // App name does not match what is registered — fix configuration
      break;
    default:
      // Show generic error message
      console.error(error.message);
  }
}
```

`error.isRetryable()` returns `true` for liveness failures, camera issues, and network errors.

## Vite dev runtime notes

### Blank screen fix (process is not defined)

If your app shows a blank page in `npm run dev` and browser console shows `ReferenceError: process is not defined` from `@synapser-sdk-distribution/entry-web-sdk`, add a `define` fallback in Vite config:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  define: {
    process: {
      env: {},
    },
  },
});
```

This avoids a runtime crash before React mounts and resolves the blank screen in dev mode.

### Liveness failure fix (Buffer is not defined / global is not defined)

The Entry SDK's AWS dependencies reference Node.js globals (`global`, `Buffer`) that don't exist in browsers. Two fixes are needed:

**1. Add `global` to Vite's `define` (compile-time substitution — replaces `global` everywhere in the bundle):**

```ts
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  define: {
    global: 'globalThis',
    process: { env: {} },
  },
});
```

**2. Polyfill `Buffer` in your entry point** (`src/main.tsx` for React), before any SDK code runs at runtime:

```typescript
import { Buffer } from 'buffer';
globalThis.Buffer = Buffer;

// ... rest of imports
```

> `global` must be handled via Vite `define` — not a `globalThis.global = globalThis` statement in source. ES module imports are hoisted, so any statement before an `import` runs too late to help the packages being imported.

## Security

- **Never commit the GitHub PAT** — use `${GITHUB_PAT_READ_PACKAGES}` in `~/.npmrc` (an environment variable or CI secret). A hard-coded PAT in `.npmrc`, source code, or `.env` files committed to version control is a critical credential leak.
- **Never log the user object** — `identifyUser()` returns PII (name, ID). Do not pass it to `console.log`, analytics, or error tracking (e.g. Sentry) without stripping sensitive fields.
- **Do not store tokens in `localStorage`** — if your app uses session tokens from Entry, store them in `sessionStorage` or an in-memory variable. `localStorage` is accessible to any JS on the page.
- **Use the Test environment during development** — never point a dev or CI environment at `EntryApiEnvironment.Live`. This prevents polluting production face data and avoids accidental live billing.
- **Serve over HTTPS in production** — browsers block camera access on non-secure origins. `localhost` is the only exception.

## Common mistakes — do not do these

- **Do not call the Entry API directly** — the SDK is the integration point. There are no raw API calls to make.
- **Do not hard-code the GitHub PAT** in `~/.npmrc`, source code, or version control. Use `${GITHUB_PAT_READ_PACKAGES}` via env.
- **Do not use `.Live` during development** — always use `.Test` first to avoid affecting production data.
- **Do not construct `new EntrySDK()`** — always use `EntrySDK.getInstance()`.
- **Do not catch errors as `Error`** — use `instanceof EntrySDKError` to access `.code` and `.isRetryable()`.
- **Do not run liveness on a machine with no camera** — the SDK requires camera access. It will not work in Node.js or SSR context.
- **Do not serve the page over HTTP in production** — browsers block camera access on non-secure origins (HTTPS required; `localhost` is the only exception).

## Supported browsers

Chrome 80+, Firefox 75+, Safari 13+, Edge 80+. Physical camera required.

## Links

- Full integration guide: https://synapser-limited.github.io/entry-sdk-docs/integration/web/
- Client onboarding: https://synapser-limited.github.io/entry-sdk-docs/getting-started/client-onboarding/
- Error handling reference: https://synapser-limited.github.io/entry-sdk-docs/integration/web-error-handling/
- Troubleshooting: https://synapser-limited.github.io/entry-sdk-docs/integration/web-troubleshooting/
