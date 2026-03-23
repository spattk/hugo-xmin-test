---
title: "Living Ahead of Time: Daylight Savings and Beyond"
date: "2024-11-27"
tags: ["security", "2fa", "totp", "time", "system-design"]
description: "What happens when you manually adjust your clock to ignore daylight savings — and why your 2FA app stops working."
---

Once daylight savings ended on November 3, 2024, I wanted to fix my sleep schedule — so I decided not to move back the analog clock in my house. The world is mostly digital though, so I manually adjusted the time on my phone and digital watch to not respect the time change. In other words, my clock was running 1 hour ahead of the true time in my region.

When I tried signing into various accounts like Namecheap and LinkedIn using my 2FA authenticator app, I consistently failed with:

> *"Oops! The OTP code you submitted is incorrect, has already been used, or expired."*

I got frustrated and eventually reached out to Namecheap's support team, who explained that all TOTP (time-based one-time password) systems rely on synchronized time — my manual adjustment had broken that. They provided steps to sync my app's time, and that fixed it.

This got me curious about how authenticator-based OTPs actually work.

## During Initial Setup

- You scan a QR code
- This establishes a **shared secret key** between the app and the server
- The secret key is stored on your device and used to generate OTPs

## The Algorithm

Most 2FA apps use **TOTP (Time-Based One-Time Password)**, derived from **HOTP (HMAC-Based One-Time Password)**.

TOTP:
- Relies on the current device time
- Divides time into **30-second intervals**

## The Algorithm in Detail

**Step 1** — The current Unix timestamp is divided into 30-second chunks:
```
T = floor(current_time / 30)
```

**Step 2** — The app calculates an HMAC (hash-based message authentication code):
```
HMAC = HMAC_SHA1(secret_key, time_step)
```
Either SHA-1, SHA-256, or SHA-512 can be used.

**Step 3** — The resulting HMAC output is truncated to generate the OTP:
```
OTP = (truncated_HMAC & 0x7FFFFFFF) % 10^6
```
The mask `0x7FFFFFFF` ensures the OTP is always a positive number.

**Step 4** — The server generates the OTP the same way and compares it to your input. To account for slight discrepancies, the server may also accept OTPs from the previous or next 30-second window.

Since my time was wrong by a full hour, the server's OTP and my app's OTP were in completely different time windows — hence the mismatch.

---

Thanks Namecheap for the quick resolution and an accidental intro to a fun system design problem.
