---
title: "1. Before you start"
---

[← Home](README.md) · **1. Prerequisites** · [2. Audit checklist](2-audit-checklist.md) · [3. API guide](3-api-guide.md) · [4. Error codes](4-error-codes.md)

# 1. Before you start (AUA prerequisites)

Get these ready first. If one is missing, the SDK will fail at registration — and the error will not always make the reason obvious.

---

## A. From UIDAI (paperwork)

- [ ] **You are an approved AUA** (or are working under one).
- [ ] **Your AUA code** for each environment you will use (staging, pre-prod, production).
- [ ] **Your app's package name** (e.g. `com.yourbank.app`) is **registered with UIDAI** against your AUA code.
- [ ] **Your app's signing certificate hash** is registered with UIDAI. *A debug build and a release build have different hashes — register the one you are testing with.*
- [ ] **Your AUA signing key pair** (RSA). UIDAI has your public key; you keep the private key safe on your server.
- [ ] **Access to the SDK package** (GitHub Packages username + token).

> 💡 Most "registration failed" problems in the first week come from a package name or certificate hash that UIDAI does not have yet. Check this first.

---

## B. On your server (AUA backend)

You need **one small API** on your server. Its only job during registration:

1. Receive the registration payload from your app.
2. Build the approval block from it.
3. **Sign it with your AUA private key** (`SHA256withRSA`).
4. Return the signed JSON to the app.

The exact JSON format is in the [API guide → Step 2](3-api-guide.md#step-2--get-it-signed-by-your-server).

You will also need your usual **Aadhaar authentication flow** on the server to send the PID block to UIDAI after capture.

> ⚠️ **The private key must never be inside the mobile app.** The sample app signs on the phone only to keep the demo simple. Do not copy that part.

---

## C. The phone

| Need | Minimum |
|---|---|
| Android version | **Android 9 (API 28)** or newer |
| Camera | Front camera |
| Google Play Services | Installed and up to date |
| Internet | Needed for registration, first-time setup, and sending results |
| Phone state | Not rooted, not an emulator, USB debugging **off**, developer options **off** |

The SDK checks the last row by itself — see the [Audit checklist](2-audit-checklist.md).

---

## D. Your Android project

| Need | Value |
|---|---|
| `minSdk` | 28 |
| `compileSdk` | 37 (or the latest your Android Gradle Plugin supports) |
| Java / Kotlin target | 11 or above |
| Camera permission | The SDK asks the user for it during capture; you don't need to request it yourself |

Add the SDK repository in `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/UIDAITechCenter/FaceRdOfflineAuthSDK")
            credentials {
                username = "<your GitHub user>"     // keep these in local.properties,
                password = "<your GitHub token>"    // never commit them
            }
        }
    }
}
```

Add the dependency in `app/build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.uidai.faceauth:face-auth-sdk:1.2.4")   // use the latest version you were given
}
```

---

## Ready?

If every box above is ticked, go to the **[API guide →](3-api-guide.md)**
