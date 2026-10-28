[← Home](README.md) · [1. Prerequisites](1-prerequisites.md) · [2. Audit checklist](2-audit-checklist.md) · [3. API guide](3-api-guide.md) · **4. Error codes**

# 4. Error codes

Don't panic about the number of codes. **You only need to handle a few groups.** Find your code below, show the user the "Tell the user" text, and follow "What you do".

Every error comes with three things:

| Field | Example | Use it for |
|---|---|---|
| `error` (`AuthError`) | `SECURITY_CHECK_FAILED` | Deciding what to do in code |
| `errorCode` | `892` | Logs and support tickets |
| `message` | "USB Debugging is enabled" | Debugging (not always user-friendly) |

During registration you get `SDKResult.Error(code, message)` instead — same codes, as text.

---

## 🚦 The 5 groups at a glance

| Group | Codes | Who fixes it |
|---|---|---|
| 🙋 User can fix (lighting, cancelled, permission) | 7xx, 915, 731, 973 | The user — let them retry |
| 📱 Phone not allowed (rooted, debugging on…) | 860–893, 911–914 | The user — change phone settings or use another phone |
| 🌐 Network | 902–906, 429 | Retry later |
| 🧑‍💻 Your integration (wrong input, setup) | 100–260, 301, 400, 401, 410, 909, 9736, 9737 | **You** — fix code or config |
| 🏛️ Server / internal | 96xx, 97xx, 98xx, 99xx, 500, 999 | Contact support with the code + txnId |

---

## 🙋 User can fix

| Code | What happened | Tell the user |
|---|---|---|
| 730, 737 | Liveness check failed | "Please move to better light and try again." |
| 732 | Too dark / too bright | "Please move to better lighting." |
| 733 | Image blurry | "Please hold the phone still." |
| 734, 735 | Background problem / screen detected | "Please check your background and try again." |
| 736 | Blink not detected | "Please blink during capture." |
| 738 | Took too long | "Time's up. Please try again." |
| 778 | Could not find good light | "Please move to better lighting and try again." |
| 731, 973 | User closed the screen | — (no message needed) |
| 915 | Camera permission denied | "Please allow camera access in Settings." |

---

## 📱 Phone not allowed

| Code | What happened | Tell the user |
|---|---|---|
| 870 – 879 | Phone is rooted / modified | "This app cannot run on this phone for security reasons." |
| 860 – 864 | App or phone software is modified | Show the SDK's message (it says exactly what to do) |
| 892 | USB debugging is on | "Please turn off USB debugging in Developer options." |
| 911 | Developer options are on | "Please turn off Developer options." |
| 912 | Running on an emulator | "Please use a real phone." |
| 913 | Camera not available | "Camera is not available. Close other camera apps and retry." |
| 914 | Hooking tool found | "This app cannot run on this phone for security reasons." |
| 880, 881 | Google Play Services is old | "Please update Google Play Services." |
| 890, 901, 907 | Low memory / low-end phone | "This phone doesn't have enough memory. Close other apps and retry." |
| 891, 893 | Android version too old | "Please update Android (version 9 or newer needed)." |

---

## 🌐 Network

| Code | What happened | What you do |
|---|---|---|
| 902 | No internet | Ask user to connect, then retry |
| 903 | Server timed out | Retry after a few seconds |
| 904 | General network problem / model download failed | Retry |
| 905 | App's security certificate expired | Update to the latest SDK version |
| 906 | Server is down for maintenance | Retry later |
| 429 | Too many requests (max 5 in 10 seconds) | Wait 10 seconds, then retry |

---

## 🧑‍💻 Your integration — fix in your code

| Code | What happened | What you do |
|---|---|---|
| **400** | `txnId` is empty | Always pass a new `txnId` |
| **401** | Phone not registered yet | Run registration first ([API guide Steps 1–3](3-api-guide.md)) |
| **410** | Registration certificate expired | Call `clearRegistration()` and register again |
| **909** | `init()` called before registration | Register first, then `init()` |
| `ALREADY_REGISTERED` | Registration called on a registered phone | Check `getState()` first, or `clearRegistration()` |
| 301 | Your server's signature is wrong | Check your AUA private key and the signed JSON format |
| 9736 | Package name not approved | Ask UIDAI to register your app's package name |
| 9737 | App certificate hash not approved | Ask UIDAI to register your signing certificate hash (debug and release are different!) |
| 9734 | Device attestation failed | Use a real, unmodified phone with up-to-date Play Services |
| 100 | `pidOptions` not valid | Leave `pidOptions` as `null` or fix the values |
| 102 | `txnId` already used | Create a new `txnId` for every attempt |
| 103 | `pidOptions.env` doesn't match environment | Use `"S"` for staging, `"PP"` for pre-prod, `"P"` for prod |
| 110 – 190, 260 | A `pidOptions` value is wrong (fType, fCount, iType, iCount, pidVer, timeout, posh, format) | Fix that field — the message names it |
| 210 | Protobuf format asked | Use XML format (default) |
| 850 | SDK version too old | Update to the latest SDK |
| 122 – 129 | Reference photo missing / unreadable / no face in it | Send a clear, front-facing photo as Base64 |
| 760 | Faces did not match (comes as `SECURITY_CHECK_FAILED`) | Not a bug — the person did not match the reference photo |

---

## 🏛️ Server / internal errors

These are rare and usually not something you can fix in code.

| Codes | Area |
|---|---|
| 9600 – 9622, 200 | Getting a secure "nonce" from the server (registration) |
| 9701 – 9742 | Device registration with server |
| 9802 – 9859 | Capture / PID creation |
| 9901 – 9943 | SDK internal |
| 500, 999, -1 | Unknown error |

**What you do:** retry once. If it keeps happening, contact support with **error code + txnId + environment + phone model + Android version**.

---

## `AuthError` values (for your `when` block)

If you'd rather handle by type than by code:

| `AuthError` | Meaning |
|---|---|
| `USER_CANCELLED` | User left the screen |
| `CAMERA_PERMISSION_DENIED` | No camera permission |
| `LIVENESS_FAILED`, `NO_FACE_DETECTED`, `MULTIPLE_FACES_DETECTED`, `POOR_IMAGE_QUALITY`, `CAPTURE_TIMEOUT` | Capture problems — let user retry |
| `SECURITY_CHECK_FAILED`, `GMS_INTEGRITY_ERROR` | Phone not allowed — **or** faces didn't match (code 760) |
| `NETWORK_ERROR` | Network or server problem |
| `MODEL_DOWNLOAD_FAILED`, `OPENCV_DOWNLOAD_FAILED`, `OPENCV_INIT_FAILED`, `SDK_INITIALIZTATION_FAILED`, `SDK_NOT_INITIALIZED` | `init()` didn't finish — retry `init()` |
| `INVALID_REFERENCE_IMAGE`, `IMAGE_RETRIEVAL_FAILED`, `EMBEDDING_FAILED` | Reference photo problem |
| `SIGNING_FAILED`, `RESPONSE_PARSING_FAILED`, `PROCESSING_ERROR` | Check the `errorCode` for details |

```kotlin
override fun onFailure(error: AuthError, errorCode: Int, message: String) {
    when (error) {
        AuthError.USER_CANCELLED          -> { /* do nothing */ }
        AuthError.NETWORK_ERROR           -> showRetry("Please check your internet and try again.")
        AuthError.SECURITY_CHECK_FAILED   ->
            if (errorCode == 760) showRetry("Face did not match. Please try again.")
            else showBlocked("This phone can't be used for face authentication.")
        else -> if (errorCode == 410) reRegister() else showRetry("Something went wrong ($errorCode).")
    }
}
```

---

[← Back to Home](README.md)
