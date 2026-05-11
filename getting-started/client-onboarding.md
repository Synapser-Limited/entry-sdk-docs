---
layout: default
title: Client Onboarding
nav_order: 3
---

# Client Onboarding

To integrate an Entry SDK, two things must be in place before your application can call the Entry API:

1. Your application must be **registered** with the Entry team.
2. Your developers must have **access to the SDK distribution packages**.

Contact [support@synapser.com](mailto:support@synapser.com) to start the process.

---

## 1. Register your application

Provide the Entry team with the following details for each platform you are integrating:

| Field               | Description                                 | Example                |
| ------------------- | ------------------------------------------- | ---------------------- |
| Application name    | Human-readable name for your app            | `Acme Mobile App`      |
| Platform            | `ios`, `android`, or `web`                  | `ios`                  |
| Bundle / package ID | Your app's unique identifier                | `com.acme.mobileapp`   |
| Allowed origins     | Web apps only — domains the SDK will run on | `https://app.acme.com` |

You need **one registration per platform** (e.g. separate registrations for your iOS app and your web app).

The Entry team will confirm when your application is registered and which environment (Test, Live) it has been activated in.

---

## 2. Get access to the SDK packages

All Entry SDKs are distributed privately through the **`synapser-sdk-distribution`** GitHub organisation.

| Platform | Package                                    | Distribution                                |
| -------- | ------------------------------------------ | ------------------------------------------- |
| iOS      | `EntrySDK`                                 | Swift Package Manager (private GitHub repo) |
| Web      | `@synapser-sdk-distribution/entry-web-sdk` | npm via GitHub Packages                     |
| Android  | `com.synapser:entry-sdk`                   | Maven via GitHub Packages                   |

Provide the Entry team with the **GitHub username** of each developer who needs access. They will receive a GitHub organisation invitation — it must be accepted before the packages can be accessed.

> If a developer does not have a GitHub account, they must create one first.

---

## 3. Developer setup — GitHub Personal Access Token

After accepting the organisation invitation, each developer creates a **GitHub Personal Access Token (PAT)** with these scopes:

- `read:packages` — download packages from GitHub Package Registry
- `repo` — access private repositories (required for iOS SPM)

Generate a PAT at: **GitHub → Settings → Developer settings → Personal access tokens**

### iOS (Swift Package Manager)

In Xcode, add the package dependency using the URL provided by the Entry team:

```
https://github.com/synapser-sdk-distribution/entry-ios-sdk-binary.git
```

Xcode will prompt for GitHub credentials. Sign in with your GitHub account (now a member of the `synapser-sdk-distribution` org).

### Web (npm)

Configure your `~/.npmrc` to route the `@synapser-sdk-distribution` scope to GitHub Packages:

```
@synapser-sdk-distribution:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=<YOUR_GITHUB_PAT>
```

Then install:

```bash
npm install @synapser-sdk-distribution/entry-web-sdk
```

### Android (Gradle / Maven)

Add the repository and credentials to your project-level `settings.gradle.kts`:

```kotlin
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/synapser-sdk-distribution/entry-android-sdk")
        credentials {
            username = "<YOUR_GITHUB_USERNAME>"
            password = "<YOUR_GITHUB_PAT>"
        }
    }
}
```

Then add the dependency:

```kotlin
dependencies {
    implementation("com.synapser:entry-sdk:<version>")
}
```

---

## Next steps

Once your application is registered and you have SDK access, follow the integration guide for your platform:

- [iOS SDK Integration](../integration/ios.md)
- [Android SDK Integration](../integration/android.md)
- [Web SDK Integration](../integration/web.md)
