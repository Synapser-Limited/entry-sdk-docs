---
layout: default
title: Web Troubleshooting
nav_order: 6
---

# Web SDK — Troubleshooting

Common issues and solutions for the Entry Web SDK.

> **Before troubleshooting, confirm:**
> - The page is served over **HTTPS** (required for camera access)
> - Your app name is registered with the Entry team
> - You are using a supported browser: Chrome 80+, Firefox 75+, Safari 13+, Edge 80+

---

## Camera not working

**Symptoms:** `CAMERA_ACCESS_DENIED` error, black screen, browser never prompts for camera permission.

**Solutions:**

1. **Check HTTPS** — Camera access requires `https://`. Local development on `localhost` is the only exception.
2. **Check browser permissions** — Click the lock icon in the address bar → Site settings → Camera → Allow.
3. **Close other camera apps** — Another application may be holding the camera.
4. **Try incognito mode** — Browser extensions can block camera access.

```typescript
// Pre-flight camera check before invoking the SDK
async function isCameraAvailable(): Promise<boolean> {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    stream.getTracks().forEach(track => track.stop());
    return true;
  } catch {
    return false;
  }
}
```

---

## Liveness check failing

**Symptoms:** `LIVENESS_CHECK_FAILED` error — user passes camera but fails liveness.

**Solutions:**

1. **Improve lighting** — Even front-facing light; avoid strong backlight.
2. **Remove obstructions** — Remove glasses, hats, masks where possible.
3. **Stay centred and still** — Keep face in the frame guide.
4. **Clean the camera lens.**

> The liveness check uses AWS Rekognition (iBeta Level 2 certified). It is designed to reject photos and recorded videos.

---

## Network errors

**Symptoms:** `NETWORK_ERROR` or `TIMEOUT_ERROR`, SDK hangs during initialisation.

**Solutions:**

1. **Check connectivity** — Verify internet access.
2. **Check firewall/proxy** — Ensure these domains are reachable: `*.amazonaws.com`, `*.entrymfa.com`.
3. **Implement retry logic** — See the [Error Handling guide](web-error-handling.md) for exponential backoff patterns.

```typescript
if (!navigator.onLine) {
  showError('Please check your internet connection');
  return;
}
```

---

## Invalid configuration

**Symptoms:** `INVALID_CONFIGURATION` or `INVALID_APP_NAME` on `getInstance()`.

**Solutions:**

1. **Verify the app name** — Must match exactly what the Entry team registered.
2. **Check environment** — Confirm you are pointing to `Test` or `Live` as appropriate.
3. **Check domain whitelist** — Your domain must be registered with the Entry team.

Contact [support@synapser.com](mailto:support@synapser.com) to verify your app configuration.

---

## User not found

**Symptoms:** `USER_NOT_FOUND` when `registerIfNotFound` is `false`.

This is expected behaviour when the user hasn't registered yet. Either:

- Set `registerIfNotFound: true` to allow registration during the same flow.
- Direct the user to a separate registration step first.

```typescript
try {
  await sdk.identifyUser(false, container);
} catch (error) {
  if (error instanceof EntrySDKError && error.code === EntrySDKErrorCode.USER_NOT_FOUND) {
    // Fall through to registration
    await sdk.identifyUser(true, container);
  }
}
```

---

## Framework-specific issues

### React: component unmounts during flow

**Problem:** The SDK overlay disappears unexpectedly.

**Solution:** Keep the container element always mounted; use `display` to show/hide it.

```tsx
// ❌ Wrong — container may unmount mid-flow
{showAuth && <div id="auth-container" />}

// ✅ Correct — container always in the DOM
<div id="auth-container" style={{ display: showAuth ? 'block' : 'none' }} />
```

### React: multiple SDK instances

**Problem:** `SDK already initialized with app 'X'` error.

**Solution:** The SDK is a singleton. Use the same app name across the component tree, or call `EntrySDK.reset()` before re-initialising.

```typescript
// ✅ Reset before switching app
EntrySDK.reset();
const sdk = EntrySDK.getInstance('new-app', env);
```

### Vue: reactivity not updating

**Problem:** User data returned from the SDK doesn't trigger Vue reactivity.

**Solution:** Assign the result to a `ref`.

```vue
<script setup>
import { ref } from 'vue';
const user = ref(null);

async function authenticate() {
  user.value = await sdk.identifyUser(true, container); // triggers reactivity
}
</script>
```

---

## Debugging

```typescript
// Enable debug logging (Test environment only)
const sdk = EntrySDK.getInstance('your-app', EntryApiEnvironment.Test, {
  enableDebugLogging: true,
});

// Full error details
catch (error) {
  if (error instanceof EntrySDKError) {
    console.log('Code:', error.code);
    console.log('Message:', error.message);
    console.log('Context:', error.context);
    console.log('Cause:', error.cause);
    console.log('Timestamp:', error.timestamp);
    console.log('Retryable:', error.isRetryable());
  }
}
```

**Browser DevTools tips:**
- **Console** — Check for SDK error messages and warnings
- **Network** — Verify API calls complete successfully
- **Application → Session Storage** — Check for `browserSessionId`

---

## FAQ

**Is HTTPS required?**
Yes. Camera access requires a secure context. `localhost` is the only exception during development.

**Which browsers are supported?**

| Browser | Minimum version |
| ------- | --------------- |
| Chrome  | 80+             |
| Firefox | 75+             |
| Safari  | 13+             |
| Edge    | 80+             |

**Can I use this in a mobile app?**
The Web SDK is for browser-based applications. For native mobile apps see the [iOS](ios.md) and [Android](android.md) integration guides.

**Can users authenticate from multiple devices?**
Yes. Each device gets a unique `deviceId`. The same user can authenticate from multiple devices.

**How do I delete a user's data (GDPR right to erasure)?**

```typescript
await sdk.deleteUser(user.entryUserId);
```

**How do I handle unsupported devices gracefully?**

```typescript
catch (error) {
  if (error.code === EntrySDKErrorCode.DEVICE_NOT_SUPPORTED ||
      error.code === EntrySDKErrorCode.BROWSER_NOT_SUPPORTED) {
    showFallbackAuthentication();
  }
}
```
