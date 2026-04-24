---
layout: default
title: Quick Reference
nav_order: 3
---

# Web SDK — Quick Reference

Essential snippets and lookup tables.

## Installation

```bash
# Configure npm for GitHub Packages (once per machine)
echo "@synapser-sdk-distribution:registry=https://npm.pkg.github.com" >> ~/.npmrc
echo "//npm.pkg.github.com/:_authToken=YOUR_GITHUB_PAT" >> ~/.npmrc

# Install
npm install @synapser-sdk-distribution/entry-web-sdk
```

---

## Initialise SDK

```typescript
import { EntrySDK, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

const sdk = EntrySDK.getInstance('your-app-name', EntryApiEnvironment.Live);
```

---

## Core methods

### Identify user (with registration)

```typescript
const user = await sdk.identifyUser(true, document.getElementById('container'));
```

### Identify user (without registration)

```typescript
const user = await sdk.identifyUser(false, document.getElementById('container'));
```

### Delete user

```typescript
await sdk.deleteUser(userId);
```

---

## Environments

| Value                      | Use for      |
| -------------------------- | ------------ |
| `EntryApiEnvironment.Test` | Testing & QA |
| `EntryApiEnvironment.Demo` | Demos & POCs |
| `EntryApiEnvironment.Live` | Production   |

---

## Error handling

```typescript
import { EntrySDKError, EntrySDKErrorCode } from '@synapser-sdk-distribution/entry-web-sdk';

try {
  const user = await sdk.identifyUser(true, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    console.error(`[${error.code}] ${error.message}`);
    if (error.isRetryable()) { /* retry */ }
  }
}
```

### Common codes

```typescript
switch (error.code) {
  case EntrySDKErrorCode.USER_NOT_FOUND:       // User needs to register
  case EntrySDKErrorCode.CAMERA_ACCESS_DENIED: // Show camera instructions
  case EntrySDKErrorCode.LIVENESS_CHECK_FAILED:// Allow retry with tips
  case EntrySDKErrorCode.NETWORK_ERROR:        // Check connection
}
```

---

## Error codes reference

### User errors

| Code                  | Retryable | Description            |
| --------------------- | --------- | ---------------------- |
| `USER_NOT_FOUND`      | ❌         | User doesn't exist     |
| `USER_ALREADY_EXISTS` | ❌         | Duplicate registration |
| `USER_CANCELLED`      | ✅         | User cancelled flow    |

### Biometric errors

| Code                      | Retryable | Description               |
| ------------------------- | --------- | ------------------------- |
| `LIVENESS_CHECK_FAILED`   | ✅         | Liveness detection failed |
| `FACE_MATCH_FAILED`       | ✅         | Face doesn't match        |
| `MULTIPLE_FACES_DETECTED` | ✅         | More than one face        |
| `NO_FACE_DETECTED`        | ✅         | No face in frame          |

### Permission errors

| Code                   | Retryable | Description                 |
| ---------------------- | --------- | --------------------------- |
| `CAMERA_ACCESS_DENIED` | ✅         | Camera permission denied    |
| `PERMISSION_DENIED`    | ❌         | Feature not allowed for app |

### Network errors

| Code            | Retryable | Description       |
| --------------- | --------- | ----------------- |
| `NETWORK_ERROR` | ✅         | Connection failed |
| `TIMEOUT_ERROR` | ✅         | Request timed out |
| `API_ERROR`     | ⚠️         | Server error      |

### Configuration errors

| Code                    | Retryable | Description              |
| ----------------------- | --------- | ------------------------ |
| `INVALID_CONFIGURATION` | ❌         | Bad SDK config           |
| `INVALID_APP_NAME`      | ❌         | App not registered       |
| `INVALID_PARAMETER`     | ❌         | Invalid method parameter |
| `INITIALIZATION_FAILED` | ❌         | SDK init failed          |

### Session errors

| Code              | Retryable | Description       |
| ----------------- | --------- | ----------------- |
| `SESSION_EXPIRED` | ✅         | Session timed out |
| `INVALID_SESSION` | ❌         | Session corrupted |

---

## `EntryUser` object

```typescript
interface EntryUser {
  entryUserId: string;
  firstName: string;
  lastName: string;
  emailAddress: string;
  mobileNumber: string;
  dateOfBirth: string;                   // ISO 8601
  gender: string;                        // "M" or "F"
  nationalityCountryCodeIso: string;
  photoIdentityDocumentType: string;
  photoIdentityDocumentNumber: string;
  deviceId: string;
  isRegistrationComplete: boolean;
}
```

---

## HTML setup

```html
<body>
  <!-- your app content -->
  <div id="entry-container"></div>
</body>
```

---

## React quick start

```tsx
import { EntrySDK, EntryApiEnvironment, EntryUser, EntrySDKError } from '@synapser-sdk-distribution/entry-web-sdk';
import { useRef, useState } from 'react';

function AuthButton() {
  const containerRef = useRef<HTMLDivElement>(null);
  const [user, setUser] = useState<EntryUser | null>(null);

  const handleAuth = async () => {
    try {
      const sdk = EntrySDK.getInstance('my-app', EntryApiEnvironment.Live);
      const result = await sdk.identifyUser(true, containerRef.current!);
      setUser(result);
    } catch (error) {
      if (error instanceof EntrySDKError) alert(error.message);
    }
  };

  return (
    <>
      <button onClick={handleAuth}>Sign In with Face</button>
      <div ref={containerRef} />
      {user && <p>Welcome, {user.firstName}!</p>}
    </>
  );
}
```

---

## Requirements checklist

- [ ] HTTPS enabled (required for camera)
- [ ] App name registered with the Entry team
- [ ] Domain whitelisted
- [ ] GitHub PAT with `read:packages` scope configured in `~/.npmrc`
- [ ] Node.js 18+
- [ ] Supported browser (Chrome 80+, Firefox 75+, Safari 13+, Edge 80+)
- [ ] Container element present in the DOM
