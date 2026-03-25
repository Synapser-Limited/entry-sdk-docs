---
layout: default
title: ERROR HANDLING
nav_order: 6
---

# Error Handling Guide

## Overview

The Entry Web SDK uses a comprehensive error handling system with typed error codes that allows consuming applications to programmatically handle different error scenarios. All public SDK methods throw `EntrySDKError` instances instead of generic `Error` objects.

## Error Structure

### EntrySDKError Class

All errors thrown by the SDK extend the custom `EntrySDKError` class, which includes:

```typescript
class EntrySDKError extends Error {
  code: EntrySDKErrorCode;        // Standardized error code
  message: string;                 // Human-readable error message
  statusCode?: number;             // HTTP status code (for API errors)
  context?: Record<string, any>;   // Additional context data
  retryable: boolean;              // Whether the operation can be retried
  cause?: Error;                   // Original error that caused this error
  timestamp: Date;                 // When the error occurred
}
```

### Methods

- `isRetryable(): boolean` - Check if the error can be retried
- `isCode(code: EntrySDKErrorCode): boolean` - Check if error matches a specific code
- `toJSON()` - Serialize error to JSON
- `toString()` - Get formatted string representation

## Error Codes

### User-Related Errors

| Code                     | Description                                  | Retryable | Typical Cause                                    |
|--------------------------|----------------------------------------------|-----------|--------------------------------------------------|
| `USER_NOT_FOUND`         | User doesn't exist and registration disabled | No        | User attempted to identify but hasn't registered |
| `USER_ALREADY_EXISTS`    | User already exists in the system            | No        | Duplicate registration attempt                   |
| `USER_VALIDATION_FAILED` | User data validation failed                  | No        | Invalid or incomplete user information           |

### Biometric Errors

| Code                      | Description                           | Retryable | Typical Cause                                         |
|---------------------------|---------------------------------------|-----------|-------------------------------------------------------|
| `LIVENESS_CHECK_FAILED`   | Biometric liveness detection failed   | Yes       | Face not detected, poor lighting, or spoofing attempt |
| `FACE_MATCH_FAILED`       | Face match confidence below threshold | Yes       | Different person or poor image quality                |
| `MULTIPLE_FACES_DETECTED` | Multiple faces when only one expected | Yes       | Multiple people in frame                              |
| `NO_FACE_DETECTED`        | No face detected in image             | Yes       | Face not visible or obscured                          |

### Permission Errors

| Code                     | Description                      | Retryable | Typical Cause                             |
|--------------------------|----------------------------------|-----------|-------------------------------------------|
| `CAMERA_ACCESS_DENIED`   | User denied camera permissions   | Yes       | Browser permissions blocked               |
| `LOCATION_ACCESS_DENIED` | User denied location permissions | Yes       | Browser permissions blocked (if required) |

### Network & API Errors

| Code                  | Description                 | Retryable | Typical Cause                             |
|-----------------------|-----------------------------|-----------|-------------------------------------------|
| `NETWORK_ERROR`       | Network connection failed   | Yes       | No internet connection or network timeout |
| `TIMEOUT_ERROR`       | API request timeout         | Yes       | Slow network or server overload           |
| `API_ERROR`           | API returned error response | Maybe     | Server error or invalid request           |
| `RATE_LIMIT_EXCEEDED` | Too many requests           | Yes       | Rate limit hit                            |

### Configuration Errors

| Code                    | Description                | Retryable | Typical Cause               |
|-------------------------|----------------------------|-----------|-----------------------------|
| `INVALID_CONFIGURATION` | SDK configuration invalid  | No        | Missing or malformed config |
| `INVALID_ENVIRONMENT`   | Invalid API environment    | No        | Wrong environment specified |
| `INVALID_APP_NAME`      | Invalid application name   | No        | App name not registered     |
| `INVALID_PARAMETER`     | Invalid parameter provided | No        | Wrong type or format        |

### Device & Browser Errors

| Code                       | Description                        | Retryable | Typical Cause                  |
|----------------------------|------------------------------------|-----------|--------------------------------|
| `DEVICE_NOT_SUPPORTED`     | Device doesn't meet requirements   | No        | Old device or missing features |
| `BROWSER_NOT_SUPPORTED`    | Browser doesn't support features   | No        | Old browser version            |
| `INVALID_ENVIRONMENT_TYPE` | Running in non-browser environment | No        | Used in Node.js/SSR            |

### User Action Errors

| Code              | Description                  | Retryable | Typical Cause            |
|-------------------|------------------------------|-----------|--------------------------|
| `USER_CANCELLED`  | User cancelled the process   | Yes       | User clicked cancel/back |
| `SESSION_EXPIRED` | Session expired              | Yes       | Too much time elapsed    |
| `INVALID_SESSION` | Session invalid or corrupted | No        | Session data corrupted   |

### Encryption Errors

| Code                   | Description                 | Retryable | Typical Cause             |
|------------------------|-----------------------------|-----------|---------------------------|
| `ENCRYPTION_FAILED`    | Encryption operation failed | No        | Crypto API unavailable    |
| `DECRYPTION_FAILED`    | Decryption operation failed | No        | Invalid encrypted data    |
| `CRYPTO_NOT_AVAILABLE` | Crypto API not available    | No        | Non-secure context (HTTP) |

### General Errors

| Code                    | Description               | Retryable | Typical Cause              |
|-------------------------|---------------------------|-----------|----------------------------|
| `NOT_INITIALIZED`       | SDK not initialized       | No        | Used before initialization |
| `INITIALIZATION_FAILED` | SDK initialization failed | No        | Config or network error    |
| `UNKNOWN_ERROR`         | Unknown error occurred    | No        | Unexpected error           |
| `INTERNAL_ERROR`        | Internal SDK error        | No        | Bug in SDK code            |

## Usage Examples

### Basic Error Handling

```typescript
import { EntrySDK, EntrySDKError, EntrySDKErrorCode, EntryApiEnvironment } from '@synapser-sdk-distribution/entry-web-sdk';

const sdk = EntrySDK.getInstance('MyApp', EntryApiEnvironment.Live);

try {
  const user = await sdk.identifyUser(true, document.getElementById('container'));
  console.log('User identified:', user);
} catch (error) {
  if (error instanceof EntrySDKError) {
    console.error(`Error [${error.code}]:`, error.message);
    console.error('Retryable:', error.isRetryable());
  }
}
```

### Handling Specific Error Codes

```typescript
try {
  const user = await sdk.identifyUser(false, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    switch (error.code) {
      case EntrySDKErrorCode.USER_NOT_FOUND:
        // Prompt user to register
        showRegistrationPrompt();
        break;
        
      case EntrySDKErrorCode.CAMERA_ACCESS_DENIED:
        // Show instructions to enable camera
        showCameraPermissionInstructions();
        break;
        
      case EntrySDKErrorCode.LIVENESS_CHECK_FAILED:
        // Allow retry with better instructions
        showLivenessTips();
        retryIdentification();
        break;
        
      case EntrySDKErrorCode.NETWORK_ERROR:
        // Show offline message
        showOfflineMessage();
        break;
        
      default:
        // Generic error handling
        showGenericError(error.message);
    }
  }
}
```

### Retry Logic for Retryable Errors

```typescript
async function identifyUserWithRetry(
  sdk: EntrySDK, 
  container: HTMLElement, 
  maxRetries: number = 3
): Promise<EntryUser> {
  let lastError: EntrySDKError | null = null;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await sdk.identifyUser(true, container);
    } catch (error) {
      if (error instanceof EntrySDKError) {
        lastError = error;
        
        // Only retry if error is retryable
        if (!error.isRetryable()) {
          throw error;
        }
        
        // Don't retry on last attempt
        if (attempt === maxRetries) {
          throw error;
        }
        
        // Wait before retrying (exponential backoff)
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Attempt ${attempt} failed. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  
  throw lastError;
}
```

### User-Friendly Error Messages

```typescript
function getUserFriendlyMessage(error: EntrySDKError): string {
  const messages: Record<EntrySDKErrorCode, string> = {
    [EntrySDKErrorCode.USER_NOT_FOUND]: 
      'We couldn\'t find your account.',
    
    [EntrySDKErrorCode.CAMERA_ACCESS_DENIED]: 
      'Camera access is required. Please enable camera permissions in your browser settings.',
    
    [EntrySDKErrorCode.LIVENESS_CHECK_FAILED]: 
      'We couldn\'t verify your identity. Please ensure your face is clearly visible and try again.',
    
    [EntrySDKErrorCode.NETWORK_ERROR]: 
      'Connection lost. Please check your internet connection and try again.',
    
    [EntrySDKErrorCode.DEVICE_NOT_SUPPORTED]: 
      'Your device doesn\'t support this feature. Please use a different device.',
    
    [EntrySDKErrorCode.USER_CANCELLED]: 
      'Authentication cancelled. Would you like to try again?',
    
    // Add more as needed
  };
  
  return messages[error.code] || error.message;
}

// Usage
try {
  await sdk.identifyUser(true, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    showToast(getUserFriendlyMessage(error), 'error');
  }
}
```

### Logging Errors with Context

```typescript
try {
  await sdk.identifyUser(true, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    // Log to your analytics service
    analytics.track('sdk_error', {
      error_code: error.code,
      error_message: error.message,
      retryable: error.retryable,
      status_code: error.statusCode,
      timestamp: error.timestamp,
      context: error.context,
    });
    
    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.error('SDK Error:', error.toJSON());
    }
  }
}
```

### React Error Boundary Integration

```typescript
import React from 'react';
import { EntrySDKError, EntrySDKErrorCode } from '@synapser-sdk-distribution/entry-web-sdk';

class EntrySDKErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { error: EntrySDKError | null }
> {
  state = { error: null };

  static getDerivedStateFromError(error: Error) {
    if (error instanceof EntrySDKError) {
      return { error };
    }
    return null;
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    if (error instanceof EntrySDKError) {
      console.error('Entry SDK Error:', {
        code: error.code,
        message: error.message,
        retryable: error.retryable,
        errorInfo
      });
    }
  }

  render() {
    if (this.state.error) {
      return (
        <div className="error-container">
          <h2>Authentication Error</h2>
          <p>{getUserFriendlyMessage(this.state.error)}</p>
          {this.state.error.isRetryable() && (
            <button onClick={() => this.setState({ error: null })}>
              Try Again
            </button>
          )}
        </div>
      );
    }

    return this.props.children;
  }
}
```

### TypeScript Type Guards

```typescript
function isCameraError(error: unknown): error is EntrySDKError {
  return (
    error instanceof EntrySDKError &&
    error.code === EntrySDKErrorCode.CAMERA_ACCESS_DENIED
  );
}

function isNetworkError(error: unknown): error is EntrySDKError {
  return (
    error instanceof EntrySDKError &&
    (error.code === EntrySDKErrorCode.NETWORK_ERROR ||
     error.code === EntrySDKErrorCode.TIMEOUT_ERROR)
  );
}

// Usage
try {
  await sdk.identifyUser(true, container);
} catch (error) {
  if (isCameraError(error)) {
    // Handle camera-specific error
    requestCameraPermission();
  } else if (isNetworkError(error)) {
    // Handle network-specific error
    showOfflineUI();
  }
}
```

## Best Practices

1. **Always check for `EntrySDKError`** - Use `instanceof` to verify error type
2. **Use error codes for logic** - Don't parse error messages, use `error.code`
3. **Respect retryable flag** - Only retry errors marked as retryable
4. **Provide user-friendly messages** - Map error codes to helpful messages
5. **Log error context** - Use `error.context` for debugging
6. **Handle specific cases** - Use switch statements for different error codes
7. **Implement fallbacks** - Always have a default error handler
8. **Track errors** - Send error metrics to your analytics service

## Common Patterns

### Retry with User Feedback

```typescript
async function identifyWithFeedback(sdk: EntrySDK, container: HTMLElement) {
  const maxAttempts = 3;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      showStatus(`Attempt ${attempt} of ${maxAttempts}...`);
      return await sdk.identifyUser(true, container);
    } catch (error) {
      if (error instanceof EntrySDKError) {
        if (attempt < maxAttempts && error.isRetryable()) {
          showStatus(`${getUserFriendlyMessage(error)} Retrying...`);
          await delay(2000);
          continue;
        }
        throw error;
      }
    }
  }
}
```

### Graceful Degradation

```typescript
async function authenticateUser(sdk: EntrySDK, container: HTMLElement) {
  try {
    // Try biometric authentication
    return await sdk.identifyUser(true, container);
  } catch (error) {
    if (error instanceof EntrySDKError) {
      if (error.code === EntrySDKErrorCode.CAMERA_ACCESS_DENIED ||
          error.code === EntrySDKErrorCode.DEVICE_NOT_SUPPORTED) {
        // Fall back to alternative authentication
        return await fallbackAuthentication();
      }
    }
    throw error;
  }
}
```

## Migration from Generic Errors

If you're upgrading from a previous SDK version that used generic `Error` objects:

**Before:**

```typescript
try {
  await sdk.identifyUser(true, container);
} catch (error) {
  if (error.message.includes('camera')) {
    // Handle camera error
  }
}
```

**After:**

```typescript
try {
  await sdk.identifyUser(true, container);
} catch (error) {
  if (error instanceof EntrySDKError) {
    if (error.code === EntrySDKErrorCode.CAMERA_ACCESS_DENIED) {
      // Handle camera error
    }
  }
}
```

## Support

For additional support or to report error handling issues, please contact support or open an issue in the repository.
