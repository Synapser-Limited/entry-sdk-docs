# Entry SDK — Integration Checklist

Use this checklist before going live with an Entry integration. It can be run by a developer, a QA engineer, or pasted into an AI assistant for automated review.

---

## Prerequisites

- [ ] Application name is registered with the Entry team (<support@synapser.com>)
- [ ] Application's bundle/package ID or web domain is registered with the Entry team
- [ ] Developer GitHub accounts are members of the `synapser-sdk-distribution` org
- [ ] GitHub Personal Access Token (PAT) has `read:packages` scope

---

## Installation

- [ ] **Web:** `~/.npmrc` routes `@synapser-sdk-distribution` scope to `https://npm.pkg.github.com`
- [ ] **Web:** `GITHUB_PAT_READ_PACKAGES` is set via environment variable or secret manager — not hard-coded
- [ ] **iOS:** SPM package added in Xcode using the URL from the Entry team
- [ ] **Android:** Maven repository block added to `settings.gradle.kts` with credentials from `~/.gradle/gradle.properties`
- [ ] Package installs/resolves without errors

---

## Permissions and platform requirements

- [ ] **iOS:** `NSCameraUsageDescription` key is present in `Info.plist`
- [ ] **Android:** `CAMERA` permission declared in `AndroidManifest.xml`
- [ ] **Web:** Application is served over HTTPS (not HTTP) in production
- [ ] Integration is being tested on a **physical device** (not a simulator or emulator)

---

## SDK initialisation

- [ ] SDK is configured/initialised **once** at app startup — not before each call
  - Web: `EntrySDK.getInstance('app-name', EntryApiEnvironment.Test)` called once
  - iOS: `EntrySDK.shared.configure(appName:environment:)` called once in `AppDelegate` or app init
  - Android: `EntrySDK.configure(context, appName, environment)` called once in `Application.onCreate()`
- [ ] App name in the SDK configuration **exactly matches** the name registered with the Entry team
- [ ] Environment is set to **Test** (not Live) for development and QA

---

## Integration

- [ ] The SDK is mounted/invoked via the SDK method — there are **no direct calls to the Entry API**
- [ ] A container `<div id="entry-container">` exists and is not hidden or obscured (Web only)
- [ ] **Web (Vite):** `npm run dev` does not crash with `ReferenceError: process is not defined` (blank screen). If it does, set a Vite `define` fallback for `process.env`
- [ ] **Web:** Liveness detection does not fail with `ReferenceError: Buffer is not defined` or `ReferenceError: global is not defined`. Fix: add `global: 'globalThis'` to Vite `define` config, and add `import { Buffer } from 'buffer'; globalThis.Buffer = Buffer;` at the top of the entry point. Do **not** use `globalThis.global = globalThis` as a source statement — ES module imports are hoisted and it runs too late.
- [ ] `identifyUser()` is called with the correct `registerIfNotFound` value for the use case:
  - `true` — identify, and register automatically if the user is not found
  - `false` — identify only (returns error if user is not registered)
- [ ] The result (identified user object) is handled — the app does something useful with it

---

## Error handling

- [ ] Errors are caught and checked as `EntrySDKError` (Web / iOS / Android) — not as generic `Error` or `Exception`
- [ ] `LIVENESS_CHECK_FAILED` is handled — user sees a helpful message and can retry
- [ ] `CAMERA_ACCESS_DENIED` is handled — user is shown how to re-enable camera permission
- [ ] `USER_CANCELLED` is handled — app returns to a sensible state (not a blank or broken screen)
- [ ] `USER_NOT_FOUND` is handled — relevant when `registerIfNotFound: false`
- [ ] `NETWORK_ERROR` is handled — user sees an appropriate offline message

---

## Environment

- [ ] **Test** environment verified: complete a liveness check against the Test environment
- [ ] Production build uses `EntryApiEnvironment.Live` / `EntryEnvironment.LIVE` / `.live`
- [ ] No `Development` or `Demo` environment values used in production builds

---

## Security

- [ ] GitHub PAT / credentials are **not** committed to version control
  - Web: `GITHUB_PAT_READ_PACKAGES` set via environment variable, not hard-coded in `.npmrc` or source
  - Android: credentials are in `~/.gradle/gradle.properties` (outside the project directory)
  - iOS: PAT stored in Xcode credential manager or CI secret, not in any committed file
- [ ] The user object returned by `identifyUser()` is **not logged** — it contains PII (name, user ID)
- [ ] Session tokens (if stored) use a secure store — **not** `localStorage` (Web), `UserDefaults` (iOS), or plain `SharedPreferences` (Android)
- [ ] The environment in the production build is `Live` — not `Test`, `Development`, or `Demo`

---

## Final verification

- [ ] A complete end-to-end flow works on a physical device against the **Test** environment:
  - [ ] Identify with registration: new user completes liveness and is registered; user object returned
  - [ ] Identify only: returning user is identified; user object returned
- [ ] Liveness failure is tested — user sees retry option, not a crash
- [ ] Camera denial is tested — user sees instructions, not a crash
- [ ] User cancellation is tested — app returns to a usable state
