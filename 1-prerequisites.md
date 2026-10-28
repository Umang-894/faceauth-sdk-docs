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
| Camera | Front camera, **5 MP or more** (sharper = better match and liveness) |
| Free memory (RAM) | About **500 MB free** — the face model needs it. Too little and `init()` fails |
| Google Play Services | Installed and up to date |
| Internet | Needed for registration, first-time setup, and sending results |
| Phone state | Not rooted, not an emulator, USB debugging **off**, developer options **off** |

The SDK checks the last row by itself — see the [Audit checklist](2-audit-checklist.md).

---

## D. Your Android project

The SDK is built and tested with the versions below. Use the same or newer in your app.

| Need | Value |
|---|---|
| `minSdk` | **28** (Android 9) |
| `compileSdk` | **36** |
| Android Gradle Plugin (AGP) | **9.1.1** |
| Kotlin | **2.3.21** |
| Gradle | **9.3.1** |
| Java / JVM target | **11** or above |
| Camera permission | The SDK asks the user for it during capture; you don't need to request it yourself |

<details markdown="1">
<summary>Where do I set these?</summary>

```toml
# gradle/libs.versions.toml
[versions]
agp = "9.1.1"
kotlin = "2.3.21"
```

```properties
# gradle/wrapper/gradle-wrapper.properties
distributionUrl=https\://services.gradle.org/distributions/gradle-9.3.1-bin.zip
```

```kotlin
// app/build.gradle.kts
android {
    compileSdk = 36
    defaultConfig { minSdk = 28 }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
}
```
</details>

### Get access to the SDK

UIDAI support gives your organisation a **GitHub username** and a **token** (with `read:packages` permission).

Put them in `local.properties` in your project's root folder:

```properties
GITHUB_USER=your_github_username
GITHUB_TOKEN=your_github_token
```

> 🔒 **Never commit this file.** Make sure your `.gitignore` has `local.properties`, `*.jks` and `*.keystore`. A leaked token lets anyone download the private SDK.

Then add the SDK repository in `settings.gradle.kts`:

```kotlin
val localProps = java.util.Properties().apply {
    val f = File(settingsDir, "local.properties")
    if (f.exists()) f.inputStream().use { load(it) }
}

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/UIDAITechCenter/FaceRdOfflineAuthSDK")
            credentials {
                username = localProps.getProperty("GITHUB_USER") ?: System.getenv("GITHUB_USER")
                password = localProps.getProperty("GITHUB_TOKEN") ?: System.getenv("GITHUB_TOKEN")
            }
        }
    }
}
```

<details markdown="1">
<summary>Using Groovy (<code>settings.gradle</code>) instead?</summary>

```groovy
def localProps = new Properties()
def f = new File(settingsDir, 'local.properties')
if (f.exists()) f.withInputStream { localProps.load(it) }

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url 'https://maven.pkg.github.com/UIDAITechCenter/FaceRdOfflineAuthSDK'
            credentials {
                username = localProps.getProperty('GITHUB_USER') ?: System.getenv('GITHUB_USER')
                password = localProps.getProperty('GITHUB_TOKEN') ?: System.getenv('GITHUB_TOKEN')
            }
        }
    }
}
```

And in `app/build.gradle`:

```groovy
implementation 'com.uidai.faceauth:face-auth-sdk:1.2.4'
```
</details>

Add the dependency in `app/build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.uidai.faceauth:face-auth-sdk:1.2.4")   // use the latest version you were given
}
```

Now run **Gradle Sync**. If it fails:

| You see | Why | Fix |
|---|---|---|
| `401 Unauthorized` | Username or token is wrong or missing | Check `local.properties`. The token needs `read:packages`. |
| `404 Not Found` | Wrong URL, or your account has no access yet | Check the URL. Ask UIDAI support to confirm your access. |
| `Could not resolve` | Gradle can't find `local.properties` | Put it in the **project root** (next to `settings.gradle.kts`), not inside `app/`. |

---

## Ready?

If every box above is ticked, go to the **[API guide →](3-api-guide.md)**
