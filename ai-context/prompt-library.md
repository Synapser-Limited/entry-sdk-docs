---
layout: default
title: Prompt Library
nav_order: 4
parent: AI Integration Context
---

# Entry SDK — Prompt Library

A set of copy-paste prompts for AI coding assistants (Cursor, Copilot, etc.). Each prompt is designed to be used **with the corresponding Entry instructions file loaded** into the assistant's context.

---

## How to use these prompts

1. Load the relevant instructions file into your AI assistant's context:
   - Web: `entry-web.instructions.md`
   - iOS: `entry-ios.instructions.md`
   - Android: `entry-android.instructions.md`
2. Copy the prompt below and paste it into the assistant
3. Review the output against the [integration checklist](integration-checklist.md) before shipping

---

## Web SDK prompts

### Add Entry biometric login to a React app

> Add Entry biometric identity verification to this React application. Install the Entry Web SDK, add an `entry-container` div to the HTML, initialise the SDK singleton with my app name and the Test environment, and add an `authenticate()` function that calls `identifyUser()` with `registerIfNotFound: true`. Wire up error handling that covers liveness failure, camera access denied, user cancelled, and user not found. Use the Test environment.

**Expected outcome:** Package install instructions, `~/.npmrc` configuration, SDK initialisation call, an HTML container div, an async authenticate function with full error handling using `EntrySDKError` and `EntrySDKErrorCode`.

---

### Mount the Entry component and handle the identification result

> Mount the Entry SDK component and handle the identification result. Assume the SDK is already installed and initialised. Show a login button that, when clicked, calls `sdk.identifyUser(true, container)`, and handles the result by displaying the returned user's name. Handle errors for liveness failure and user cancelled.

**Expected outcome:** A button, a container div, an event handler that calls `identifyUser`, a success branch that displays the user's name, and error branches for the two specified error codes.

---

### Handle liveness failure with a retry flow

> Add retry logic for Entry SDK liveness failures. When `identifyUser()` throws a `LIVENESS_CHECK_FAILED` error, show a message with lighting tips and a Retry button that calls `identifyUser()` again. Cap retries at 3. After the cap, show a fallback message.

**Expected outcome:** A retry counter, a retry button visible only on liveness failure, tips text, and a fallback state after 3 attempts.

---

### Switch the Entry SDK from Test to Live environment

> Update this integration to use the Entry Live environment instead of Test. Find the `EntrySDK.getInstance()` call and update the environment argument to `EntryApiEnvironment.Live`. Do not change any other behaviour.

**Expected outcome:** A single-line change from `EntryApiEnvironment.Test` to `EntryApiEnvironment.Live`.

---

## iOS SDK prompts

### Integrate Entry into an iOS app using SwiftUI

> Integrate Entry biometric identity verification into this SwiftUI app. Add the Entry iOS SDK via Swift Package Manager using the URL provided, add `NSCameraUsageDescription` to `Info.plist`, call `EntrySDK.shared.configure()` in the app entry point with the Test environment, and add an `authenticate()` async function that calls `identifyUser(registerIfNotFound: true, presenter:)`. Handle liveness failure, camera access denied, and user not found errors.

**Expected outcome:** SPM setup instructions, `Info.plist` key, a `configure()` call in `App` init or `AppDelegate`, and an async `authenticate()` function with typed error handling.

---

### Add Entry to a UIKit view controller

> Add Entry biometric authentication to this UIKit view controller. Assume `EntrySDK.shared.configure()` has already been called in `AppDelegate`. Add an authenticate method that calls `identifyUser(registerIfNotFound: true, presenter: self)` and handles the result. Use Swift Concurrency (async/await).

**Expected outcome:** An `authenticate()` async method with `Task { }` wrapping for UIKit context, success handling, and error handling for the key error cases.

---

## Android SDK prompts

### Integrate Entry into an Android app using Kotlin

> Integrate Entry biometric identity verification into this Android app. Configure the Gradle repository for GitHub Packages, add the `com.synapser:entry-sdk` dependency, add the CAMERA permission to `AndroidManifest.xml`, configure the SDK in the Application class with the TEST environment, and add an `authenticate()` function in MainActivity that calls `EntrySDK.identifyUser(registerIfNotFound = true, activity = this)`. Handle liveness failure, camera denied, and user not found errors.

**Expected outcome:** `settings.gradle.kts` repository block, `build.gradle.kts` dependency, manifest permission, `Application` class with `configure()`, and an authenticate coroutine function with error handling.

---

### Handle Entry errors in an Android app

> Add error handling for Entry SDK exceptions in this Android Activity. The `authenticate()` function already calls `EntrySDK.identifyUser()`. Catch `EntrySDKException` and handle these cases: `USER_NOT_FOUND` (show registration prompt), `LIVENESS_CHECK_FAILED` (show retry dialog), `CAMERA_ACCESS_DENIED` (direct to App Settings), `USER_CANCELLED` (do nothing).

**Expected outcome:** A `when` block on `e.code` with the four specified cases wired to appropriate UI actions.

---

## Notes for prompt testing

When testing a prompt:

1. Open a blank project (no Entry code present)
2. Load the instructions file into the assistant context
3. Paste the prompt exactly
4. Evaluate output against the [integration checklist](integration-checklist.md)
5. Note any hallucinations or missing steps

Known failure modes to watch for:

- AI invents a `new EntrySDK()` constructor call (should use `getInstance()` / `shared`)
- AI calls a raw Entry API endpoint directly (should never happen — SDK-only integration)
- AI uses the wrong environment for production (should be `.Live` / `LIVE`, not `.Test` / `TEST`)
- AI skips `~/.npmrc` GitHub Packages authentication (Web)
- AI skips `NSCameraUsageDescription` in `Info.plist` (iOS)
