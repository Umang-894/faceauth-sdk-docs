---
title: "2. Audit checklist"
---

[← Home](README.md) · [1. Prerequisites](1-prerequisites.md) · **2. Audit checklist** · [3. API guide](3-api-guide.md) · [4. Error codes](4-error-codes.md)

# 2. Audit checklist

Two simple lists:

1. **What the SDK already checks** — you get this for free, nothing to build.
2. **What you must take care of** — the SDK cannot do these for you, and an auditor will ask about them.

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

### 📱 Your app
- [ ] Release build uses `Environment.PROD`. Test builds never point to production.
- [ ] Release build is **not debuggable** and is **obfuscated** (R8/ProGuard on).
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

> ℹ️ This page lists what the SDK enforces and common good practice. The **official audit list comes from UIDAI and your AUA agreement** — always follow that if anything differs.

---

Next: **[API guide →](3-api-guide.md)**
