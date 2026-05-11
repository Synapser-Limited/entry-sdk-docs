---
layout: default
title: Native Liveness
nav_order: 9
parent: Integration Guides
---

# Native Liveness Integration (iOS + Android)

This guide explains how iOS and Android apps use native liveness detection with the Entry SDK. Native liveness uses the platform's native camera directly (via AWS Amplify liveness UI), bypassing the WebView camera limitations that affect web-based liveness.

## How it works

```
1. Native app creates a liveness session via the Entry API
   POST /Liveness → {sessionId, credentials}

2. Native app presents the AWS Amplify native liveness UI
   User completes face liveness check using the device camera

3. Native app passes the completed session ID to the Entry WebView bridge
   IDENTIFY_USER { registerIfNotFound: true, nativeLivenessSessionId: "abc123" }

4. Web SDK fetches the liveness result from the Entry API
   GET /Liveness/{sessionId}/result

5. Web SDK processes the result
   - User found → Return EntryUser
   - No match → Continue to registration flow
   - Low confidence → Return error

6. Web SDK sends result back to the native app
   USER_IDENTIFIED or ERROR bridge message
```

## Requirements

### iOS

- iOS 14.0+
- Xcode 14+
- Swift 5.7+
- Physical device (camera required — simulator not supported)

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/aws-amplify/amplify-ui-swift-liveness", from: "1.1.8")
]
```

### Android

- Android API 21+
- Physical device (camera required — emulator not supported for liveness)

```kotlin
// build.gradle.kts
dependencies {
    implementation("com.amplifyframework.ui:liveness:1.2.5")
    implementation("com.amplifyframework:core-kotlin:1.1.0")
    implementation("com.amplifyframework:aws-auth-cognito:2.14.11")
}
```

## Usage

### iOS (Swift)

```swift
// 1. Create a liveness session via the Entry API
let session = try await createLivenessSession(appName: "MyApp", deviceId: deviceId)

// 2. Present the native Amplify liveness UI
let livenessView = FaceLivenessDetectorView(
    sessionID: session.sessionId,
    region: "eu-west-2",
    onCompletion: { result in
        if case .success = result {
            // 3. Pass the session ID to the WebView bridge
            sendSessionToWebView(sessionId: session.sessionId)
        }
    }
)
.credentials(
    accessKeyId: session.credentials.accessKeyId,
    secretAccessKey: session.credentials.secretAccessKey,
    sessionToken: session.credentials.sessionToken
)

// 3. Bridge message to the WebView
let message: [String: Any] = [
    "id": "msg_\(UUID().uuidString)",
    "type": "IDENTIFY_USER",
    "payload": [
        "registerIfNotFound": true,
        "nativeLivenessSessionId": session.sessionId
    ],
    "timestamp": Int(Date().timeIntervalSince1970 * 1000),
    "version": "1.0.0"
]
webView.evaluateJavaScript("window.EntryBridge.handleCommand('\(jsonString)')")
```

### Android (Kotlin)

```kotlin
// 1. Create a liveness session via the Entry API
val session = createLivenessSession(appName = "MyApp", deviceId = deviceId)

// 2. Present the native Amplify liveness UI
FaceLivenessDetector(
    sessionId = session.sessionId,
    region = "eu-west-2",
    credentialsProvider = { /* use session.credentials */ },
    onComplete = {
        // 3. Pass the session ID to the WebView bridge
        sendSessionToWebView(session.sessionId)
    },
    onError = { error -> /* handle */ }
)

// 3. Bridge message to the WebView
val message = JSONObject().apply {
    put("id", "msg_${System.currentTimeMillis()}")
    put("type", "IDENTIFY_USER")
    put("payload", JSONObject().apply {
        put("registerIfNotFound", true)
        put("nativeLivenessSessionId", session.sessionId)
    })
    put("timestamp", System.currentTimeMillis())
    put("version", "1.0.0")
}
webView.evaluateJavascript("window.EntryBridge.handleCommand('$message')", null)
```

## API Reference

### Create liveness session

```
POST /api/v2.0/liveness
Headers:
  X-ClientAppPackageOrBundleId: {bundleId}
  X-ClientAppDeviceId: {deviceId}
  X-OriginOs: ios | android
  X-OriginSdkVersion: {version}

Response:
{
  "sessionId": "string",
  "credentials": {
    "accessKeyId": "string",
    "secretAccessKey": "string",
    "sessionToken": "string",
    "expiration": "string"
  }
}
```

### Bridge message: IDENTIFY_USER with native liveness

```typescript
interface IdentifyUserPayload {
  registerIfNotFound: boolean;
  nativeLivenessSessionId?: string;  // Pass the session ID from native liveness
}
```

When `nativeLivenessSessionId` is provided, the Web SDK skips its own liveness UI and uses the completed native session for identification.

## Troubleshooting

### Session ID not found

Ensure the liveness session was successfully completed before sending the session ID to the WebView. If the user cancels or fails liveness, do not send the session ID.

### Camera access denied

Add the required permission to `Info.plist` (iOS) or `AndroidManifest.xml` (Android):

**iOS:**
```xml
<key>NSCameraUsageDescription</key>
<string>Entry needs camera access for identity verification.</string>
```

**Android:**
```xml
<uses-permission android:name="android.permission.CAMERA" />
```

### Credentials expired

Liveness session credentials have a short TTL. Create the session and present the liveness UI in the same user interaction — do not cache credentials between sessions.

### Low confidence result

If the Web SDK returns a low-confidence liveness error, prompt the user to retry. Ensure lighting conditions are adequate and the camera is unobstructed.
