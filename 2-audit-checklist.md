---
title: "2. Audit checklist"
---

[← Home](README.md) · [1. Prerequisites](1-prerequisites.md) · **2. Audit checklist** · [3. API guide](3-api-guide.md) · [4. Error codes](4-error-codes.md)

# 2. Audit checklist

Three things to know:

1. **You need a security audit report first** — no report, no integration key.
2. **What the SDK already checks** — you get this for free, nothing to build.
3. **What your app must do** — the SDK cannot do these for you, and the auditor will check them.

---

## Before you get your key — Security Audit Report

Before UIDAI issues your integration key, you must submit a **Security Audit Report** of your app that includes both:

- **SAST** (Static testing) — a scan of your app's code.
- **DAST** (Dynamic testing) — a test of your running app.

The report must show that your app has the controls in [Part 2](#part-2--what-you-must-take-care-of). **Approval is held until the report is reviewed and accepted.**

> 🔁 If you later **remove a control**, **lower your Play Integrity level**, or **add a third-party SDK that loads code at runtime**, you must tell UIDAI **before** releasing that version.

---

## Part 1 — What the SDK checks for you

The phone checks run automatically during registration and every time you call `init()`. The capture checks run during every authentication. If any check fails, the SDK stops and gives you an error code.

| Check | Why it matters | Error code if it fails |
|---|---|---|
| Phone is **not rooted** | A rooted phone can fake the camera or steal data | 870 – 879 |
| **USB debugging is off** | Someone could control the phone from a computer | 892 |
| **Developer options are off** | Same reason as above | 911 |
| **Not an emulator** | Must be a real phone with a real camera | 912 |
| **No hooking tools** (Frida, Xposed, etc.) | These tools can change what the app does while it runs | 914 |
| **Camera is available** and permission given | Needed to capture the face | 913 / 915 |
| **Play Services security is up to date** | Keeps the network connection secure | 881 |
| **Device key is hardware-backed** | The phone's key is created inside secure hardware and proved to UIDAI at registration | registration fails |
| **App identity** (package name + signing certificate) | Stops a copied or modified app from registering | 9736 / 9737 |
| **Device certificate is still valid** | Expired registration must be renewed | 410 |
| **Liveness** | Real person, not a printed photo or a screen | 730 – 737 |
| **Every request has a txnId** | Each attempt can be traced | 400 |
| **Rate limit** (max 5 requests / 10 seconds) | Stops misuse and brute-force | 429 |
| **Face data is encrypted** inside the PID block | Only UIDAI can read it | — |
| **Registration data is stored encrypted** on the phone | Nobody can copy it from the phone's storage | — |

> You **cannot turn these off** in a production build. If a tester's phone fails, it is usually USB debugging or developer options — switch them off and retry.

---

## Part 2 — What *you* must take care of

### 🔑 Keys and signing
- [ ] AUA **private key lives only on your server** (an HSM is best). Never in the app, never in Git.
- [ ] Your server **checks the device's signature** on the registration payload before signing its own approval.
- [ ] Only your own logged-in users can call your signing API (it must not be an open endpoint).

### 🧾 Transactions
- [ ] Every authentication uses a **new, unique `txnId`**. Never reuse one.
- [ ] You **log every attempt** on your server: txnId, time, result, error code. Keep these logs for the period your AUA agreement asks for.

### 🙈 Personal data
- [ ] You **do not store** the PID block, the captured face image, or the reference image after the transaction is done.
- [ ] If you use `returnCapturedImage = true`, you have a clear reason for it, and the image is not saved or sent anywhere else.
- [ ] Aadhaar numbers are **not written to app logs** (Logcat) or crash reports.
- [ ] The user is told **why** their face is being captured and **agrees** before you start (consent screen).

### 🛡️ App security controls (checked in your audit report)

Your own app must have these too — not just the SDK.

| Control | In simple words |
|---|---|
| **Play Integrity** | Get a verdict each session and check it **on your server**. Minimum `MEETS_DEVICE_INTEGRITY`; `MEETS_STRONG_INTEGRITY` recommended. |
| **Root detection** | Detect rooted phones (su, Magisk, Zygisk, KernelSU) in **native (JNI) code** — Java-only checks are not enough. |
| **Debugger detection** | Release build is not debuggable, and you check at runtime that no debugger is attached. |
| **Developer options / USB debugging** | Detect both. USB debugging on during capture = fail. |
| **Hooking detection** | Detect Frida, Xposed / LSPosed, Substrate. |
| **Emulators & clones** | Block emulators, cloud phones and "dual app / parallel space" clones. |
| **App signature check** | At runtime, check your APK's signing certificate matches the real one, so a re-signed copy won't run. |
| **Screen protection** | Use `FLAG_SECURE` on capture and result screens (no screenshots or recording). Ignore taps when another app draws over yours (tapjacking). |
| **Mock location & VPN** | If your flow uses location, detect fake GPS apps and active VPN / proxy. |
| **Secure network** | TLS 1.2+, no plain HTTP (`cleartextTrafficPermitted=false`), no debug overrides, and **certificate pinning** for SDK-facing endpoints. |
| **Hard to reverse-engineer** | Code shrinking + obfuscation (R8), encrypted sensitive strings, security logic in native code. |
| **No sensitive logs or backups** | Never write face data, PID, keys or tokens to Logcat, crash reports, analytics or disk. Strip logs from release builds. Set `android:allowBackup="false"`. |

### 📱 Your app
- [ ] Release build uses `Environment.PROD`. Test builds never point to production.
- [ ] You show the user a **friendly message** for each error (see [Error codes](4-error-codes.md)) — not a raw code alone.
- [ ] When you get error **410** (registration expired), you run registration again (see [API guide](3-api-guide.md#when-registration-expires-error-410)).

### 🌐 Network
- [ ] All calls between your app and your server use **HTTPS**.
- [ ] Your server sends PID blocks to UIDAI only through your approved channel (ASA).

---

## Quick self-test before audit

Try each of these on a test phone. Each one **should fail** with the code shown:

| Do this | Expected |
|---|---|
| Turn on USB debugging, then call `init()` | 892 |
| Turn on developer options, then call `init()` | 911 |
| Run on an emulator | 912 |
| Deny camera permission | 915 |
| Call `startAuthentication` with an empty `txnId` | 400 |
| Hold up a photo of a face to the camera | Liveness failure (730 – 737) |

If all of these fail the way they should, the device-side part of your audit is in good shape.

> ℹ️ If anything here differs from your AUA agreement or the latest UIDAI guidance, **follow UIDAI**.

---

Next: **[API guide →](3-api-guide.md)**
