---
layout: default
title: Quick Reference
nav_order: 3
description: "Copy-paste code snippets and quick reference for Entry Web SDK"
---

# Quick Reference

Essential code snippets and quick lookup tables.
{: .fs-6 .fw-300 }

---

## Installation

```bash
# Configure npm for GitHub Packages
echo "@synapser-sdk-distribution:registry=https://npm.pkg.github.com" >> ~/.npmrc
echo "//npm.pkg.github.com/:_authToken=YOUR_GITHUB_TOKEN" >> ~/.npmrc

# Install
npm install @synapser-sdk-distribution/entry-web-sdk
```

---

## Basic Usage

### Initialize SDK

```typescript
import { EntrySDK, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

const sdk = EntrySDK.getInstance('your-app-name', EntryApiEnvironment.Live);
```

### Identify User (with registration)

```typescript
const user = await sdk.identifyUser(true, document.getElementById('container'));
```

### Identify User (without registration)

```typescript
const user = await sdk.identifyUser(false, document.getElementById('container'));
```

### Delete User

```typescript
await sdk.deleteUser(userId);
```

### Identify from Photo

```typescript
const users = await sdk.identifyUsersFromPhoto(base64PhotoData);
```

---

## Environments

| Environment | Value | Use For |
|-------------|-------|---------|
| `EntryApiEnvironment.Development` | Development | Local development |
| `EntryApiEnvironment.Test` | Test | Testing & QA |
| `EntryApiEnvironment.Demo` | Demo | Demos & POCs |
| `EntryApiEnvironment.Live` | Live | Production |

---

## Error Handling

### Basic Pattern

```typescript
import { EntrySDKError, EntrySDKErrorCode } from '@synapser-sdk-distribution/entry-web-sdk';

try {
  const user = await sdk.identifyUser(true, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    console.error(`[${error.code}] ${error.message}`);
    
    if (error.isRetryable()) {
      // Can retry this operation
    }
  }
}
```

### Handle Specific Errors

```typescript
switch (error.code) {
  case EntrySDKErrorCode.USER_NOT_FOUND:
    // User needs to register
    break;
  case EntrySDKErrorCode.CAMERA_ACCESS_DENIED:
    // Show camera permission instructions
    break;
  case EntrySDKErrorCode.LIVENESS_CHECK_FAILED:
    // Allow retry with tips
    break;
  case EntrySDKErrorCode.NETWORK_ERROR:
    // Check connection
    break;
}
```

---

## Error Codes Reference

### User Errors

| Code | Retryable | Description |
|------|-----------|-------------|
| `USER_NOT_FOUND` | ❌ | User doesn't exist |
| `USER_ALREADY_EXISTS` | ❌ | Duplicate registration |
| `USER_CANCELLED` | ✅ | User cancelled flow |

### Biometric Errors

| Code | Retryable | Description |
|------|-----------|-------------|
| `LIVENESS_CHECK_FAILED` | ✅ | Liveness detection failed |
| `FACE_MATCH_FAILED` | ✅ | Face doesn't match |
| `MULTIPLE_FACES_DETECTED` | ✅ | More than one face |
| `NO_FACE_DETECTED` | ✅ | No face in frame |

### Permission Errors

| Code | Retryable | Description |
|------|-----------|-------------|
| `CAMERA_ACCESS_DENIED` | ✅ | Camera permission denied |
| `PERMISSION_DENIED` | ❌ | Feature not allowed for app |

### Network Errors

| Code | Retryable | Description |
|------|-----------|-------------|
| `NETWORK_ERROR` | ✅ | Connection failed |
| `TIMEOUT_ERROR` | ✅ | Request timed out |
| `API_ERROR` | ⚠️ | Server error |

### Configuration Errors

| Code | Retryable | Description |
|------|-----------|-------------|
| `INVALID_CONFIGURATION` | ❌ | Bad SDK config |
| `INVALID_APP_NAME` | ❌ | App not registered |
| `INVALID_PARAMETER` | ❌ | Invalid method parameter |
| `INITIALIZATION_FAILED` | ❌ | SDK init failed |

### Session Errors

| Code | Retryable | Description |
|------|-----------|-------------|
| `SESSION_EXPIRED` | ✅ | Session timed out |
| `INVALID_SESSION` | ❌ | Session corrupted |

---

## EntryUser Object

```typescript
interface EntryUser {
  entryUserId: string;           // Unique ID
  firstName: string;
  lastName: string;
  emailAddress: string;
  mobileNumber: string;
  dateOfBirth: string;           // ISO 8601
  gender: string;                // "M" or "F"
  nationalityCountryCodeIso: string;
  photoIdentityDocumentType: string;
  photoIdentityDocumentNumber: string;
  deviceId: string;
  isRegistrationComplete: boolean;
}
```

---

## HTML Setup

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My App</title>
</head>
<body>
  <!-- SDK renders authentication UI here -->
  <div id="entry-container"></div>
  
  <script type="module">
    import { EntrySDK, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';
    
    const sdk = EntrySDK.getInstance('my-app', EntryApiEnvironment.Live);
    const user = await sdk.identifyUser(true, document.getElementById('entry-container'));
  </script>
</body>
</html>
```

---

## React Quick Start

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
      if (error instanceof EntrySDKError) {
        alert(error.message);
      }
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

## Requirements Checklist

- [ ] HTTPS enabled (required for camera)
- [ ] App name registered with Synapser
- [ ] Domain whitelisted
- [ ] GitHub token with `read:packages` scope
- [ ] Node.js 18+
- [ ] Supported browser (Chrome 80+, Firefox 75+, Safari 13+, Edge 80+)
- [ ] Container element in DOM
