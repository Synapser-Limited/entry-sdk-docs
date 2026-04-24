---
layout: default
title: Error Handling
nav_order: 6
---

# Web SDK — Error Handling Guide

All public SDK methods throw `EntrySDKError` instances rather than generic `Error` objects, allowing consuming applications to handle errors programmatically.

## Error structure

```typescript
class EntrySDKError extends Error {
  code: EntrySDKErrorCode;        // Standardised error code
  message: string;                // Human-readable message
  statusCode?: number;            // HTTP status code (API errors only)
  context?: Record<string, any>;  // Additional context
  retryable: boolean;             // Whether the operation can be retried
  cause?: Error;                  // Original error
  timestamp: Date;                // When the error occurred
}
```

**Methods:**
- `isRetryable(): boolean` — check if the error can be retried
- `isCode(code: EntrySDKErrorCode): boolean` — check for a specific code
- `toJSON()` — serialise to JSON
- `toString()` — formatted string representation

---

## Error codes

### User errors

| Code                     | Retryable | Description                                     |
| ------------------------ | --------- | ----------------------------------------------- |
| `USER_NOT_FOUND`         | No        | User doesn't exist and registration is disabled |
| `USER_ALREADY_EXISTS`    | No        | Duplicate registration attempt                  |
| `USER_VALIDATION_FAILED` | No        | Invalid or incomplete user information          |
| `USER_CANCELLED`         | Yes       | User cancelled the flow                         |

### Biometric errors

| Code                      | Retryable | Description                                           |
| ------------------------- | --------- | ----------------------------------------------------- |
| `LIVENESS_CHECK_FAILED`   | Yes       | Face not detected, poor lighting, or spoofing attempt |
| `FACE_MATCH_FAILED`       | Yes       | Face match confidence below threshold                 |
| `MULTIPLE_FACES_DETECTED` | Yes       | Multiple people in frame                              |
| `NO_FACE_DETECTED`        | Yes       | Face not visible or obscured                          |

### Permission errors

| Code                     | Retryable | Description                                      |
| ------------------------ | --------- | ------------------------------------------------ |
| `CAMERA_ACCESS_DENIED`   | Yes       | Browser camera permission denied                 |
| `LOCATION_ACCESS_DENIED` | Yes       | Browser location permission denied (if required) |
| `PERMISSION_DENIED`      | No        | Feature not allowed for this app                 |

### Network & API errors

| Code                  | Retryable | Description                               |
| --------------------- | --------- | ----------------------------------------- |
| `NETWORK_ERROR`       | Yes       | No internet connection or network timeout |
| `TIMEOUT_ERROR`       | Yes       | API request timeout                       |
| `API_ERROR`           | Maybe     | Server error or invalid request           |
| `RATE_LIMIT_EXCEEDED` | Yes       | Rate limit hit                            |

### Configuration errors

| Code                    | Retryable | Description                                 |
| ----------------------- | --------- | ------------------------------------------- |
| `INVALID_CONFIGURATION` | No        | Missing or malformed SDK config             |
| `INVALID_ENVIRONMENT`   | No        | Wrong environment specified                 |
| `INVALID_APP_NAME`      | No        | App name not registered with Entry team     |
| `INVALID_PARAMETER`     | No        | Wrong type or format for a method parameter |
| `INITIALIZATION_FAILED` | No        | SDK init failed                             |

### Device & browser errors

| Code                       | Retryable | Description                       |
| -------------------------- | --------- | --------------------------------- |
| `DEVICE_NOT_SUPPORTED`     | No        | Device doesn't meet requirements  |
| `BROWSER_NOT_SUPPORTED`    | No        | Browser version too old           |
| `INVALID_ENVIRONMENT_TYPE` | No        | SDK used in Node.js / SSR context |

### Session errors

| Code              | Retryable | Description            |
| ----------------- | --------- | ---------------------- |
| `SESSION_EXPIRED` | Yes       | Session timed out      |
| `INVALID_SESSION` | No        | Session data corrupted |

### Encryption errors

| Code                   | Retryable | Description               |
| ---------------------- | --------- | ------------------------- |
| `ENCRYPTION_FAILED`    | No        | Crypto API unavailable    |
| `DECRYPTION_FAILED`    | No        | Invalid encrypted data    |
| `CRYPTO_NOT_AVAILABLE` | No        | Non-secure context (HTTP) |

### General errors

| Code              | Retryable | Description                    |
| ----------------- | --------- | ------------------------------ |
| `NOT_INITIALIZED` | No        | SDK used before initialisation |
| `UNKNOWN_ERROR`   | No        | Unexpected error               |
| `INTERNAL_ERROR`  | No        | Bug in SDK code                |

---

## Usage examples

### Basic error handling

```typescript
import { EntrySDK, EntrySDKError, EntrySDKErrorCode, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

const sdk = EntrySDK.getInstance('MyApp', EntryApiEnvironment.Live);

try {
  const user = await sdk.identifyUser(true, document.getElementById('container'));
} catch (error) {
  if (error instanceof EntrySDKError) {
    console.error(`Error [${error.code}]:`, error.message);
    console.error('Retryable:', error.isRetryable());
  }
}
```

### Handling specific codes

```typescript
try {
  const user = await sdk.identifyUser(false, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    switch (error.code) {
      case EntrySDKErrorCode.USER_NOT_FOUND:
        showRegistrationPrompt();
        break;
      case EntrySDKErrorCode.CAMERA_ACCESS_DENIED:
        showCameraPermissionInstructions();
        break;
      case EntrySDKErrorCode.LIVENESS_CHECK_FAILED:
        showLivenessTips();
        break;
      case EntrySDKErrorCode.NETWORK_ERROR:
        showOfflineMessage();
        break;
      default:
        showGenericError(error.message);
    }
  }
}
```

### Retry logic with exponential backoff

```typescript
async function identifyUserWithRetry(
  sdk: EntrySDK,
  container: HTMLElement,
  maxRetries = 3
): Promise<EntryUser> {
  let lastError: EntrySDKError | null = null;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await sdk.identifyUser(true, container);
    } catch (error) {
      if (!(error instanceof EntrySDKError)) throw error;
      lastError = error;
      if (!error.isRetryable() || attempt === maxRetries) throw error;

      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}
```

### User-friendly messages

```typescript
function getUserFriendlyMessage(error: EntrySDKError): string {
  const messages: Partial<Record<EntrySDKErrorCode, string>> = {
    [EntrySDKErrorCode.USER_NOT_FOUND]:
      "We couldn't find your account.",
    [EntrySDKErrorCode.CAMERA_ACCESS_DENIED]:
      'Camera access is required. Please enable camera permissions in your browser settings.',
    [EntrySDKErrorCode.LIVENESS_CHECK_FAILED]:
      "We couldn't verify your identity. Please ensure your face is clearly visible and try again.",
    [EntrySDKErrorCode.NETWORK_ERROR]:
      'Connection lost. Please check your internet connection and try again.',
    [EntrySDKErrorCode.USER_CANCELLED]:
      'Authentication cancelled. Would you like to try again?',
  };
  return messages[error.code] ?? error.message;
}
```

### Logging with context

```typescript
try {
  await sdk.identifyUser(true, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    analytics.track('sdk_error', {
      error_code: error.code,
      retryable: error.retryable,
      status_code: error.statusCode,
      timestamp: error.timestamp,
      context: error.context,
    });

    if (process.env.NODE_ENV === 'development') {
      console.error('SDK Error:', error.toJSON());
    }
  }
}
```
