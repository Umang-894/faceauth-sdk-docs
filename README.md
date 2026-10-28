# Face Auth SDK — Integration Guide

> **In one line:** add this SDK to your Android app, and it will open the camera, check that a real person is in front of it, and give you a signed, encrypted result (PID block) that you send for Aadhaar face authentication.

You do **not** need to know face recognition, cryptography or camera code. The SDK does all of that. You only call a few functions.

---

## Pick your page

| Page | Read this if you want to know… | Time to read |
|---|---|---|
| [1. Before you start (AUA prerequisites)](1-prerequisites.md) | What you must have *before* writing any code | 3 min |
| [2. Audit checklist](2-audit-checklist.md) | What the SDK checks on the phone, and what *you* must take care of to pass audit | 4 min |
| [3. API guide](3-api-guide.md) | How to call the SDK, step by step, with copy-paste code | 8 min |
| [4. Error codes](4-error-codes.md) | What an error code means and what to do about it | look-up |

New here? Read them in order. Already integrating? Jump to page 3.

---

## The whole thing in one picture

```mermaid
sequenceDiagram
    autonumber
    participant App as 📱 Your App
    participant SDK as 🙂 Face Auth SDK
    participant AUA as 🖥️ Your AUA Server
    participant UIDAI as 🏛️ UIDAI

    rect rgba(120, 160, 255, 0.12)
    Note over App,AUA: Once per phone — Registration
    App->>SDK: initiateDeviceRegistration()
    SDK-->>App: registration payload
    App->>AUA: payload
    AUA-->>App: signed approval
    App->>SDK: completeDeviceRegistration()
    end

    rect rgba(120, 200, 140, 0.12)
    Note over App,SDK: Once per app launch
    App->>SDK: init()
    SDK-->>App: onReady
    end

    rect rgba(255, 180, 90, 0.14)
    Note over App,UIDAI: Every authentication
    App->>SDK: startAuthentication()
    Note right of SDK: camera + liveness
    SDK-->>App: PID block
    App->>AUA: PID block
    AUA->>UIDAI: Aadhaar auth request
    UIDAI-->>AUA: Yes / No
    end
```

| Colour | When it happens | Steps |
|---|---|---|
| 🔵 Blue | **Once per phone** (registration) | 1 – 5 |
| 🟢 Green | **Once per app launch** | 6 – 7 |
| 🟠 Orange | **Every time a user authenticates** | 8 – 12 |

---

## Words you will see

| Word | Simple meaning |
|---|---|
| **AUA** | You — the organisation that is allowed by UIDAI to do Aadhaar authentication. |
| **AUA code** | Your organisation's ID, given by UIDAI. |
| **Registration** | A one-time handshake that tells UIDAI "this phone + this app is trusted by this AUA". |
| **txnId** | A unique ID *you* create for every authentication attempt. Never reuse it. |
| **PID block** | The encrypted package with the captured face. Only UIDAI can open it. |
| **Liveness** | Check that a real live person is in front of the camera, not a photo or a screen. |
| **Reference image** | A known photo of the person, used when you want the SDK to match faces on the phone. |

---

## Need help?

When you contact support, always share: **the error code**, **the txnId**, **the environment** (STAGING / PRE_PROD / PROD) and **the phone model + Android version**.
