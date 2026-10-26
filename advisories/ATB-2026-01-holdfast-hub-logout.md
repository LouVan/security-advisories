# Advisory ATB-2026-01: Hub WebSocket Session Desynchronization on Logout

- **Target:** HoldFast Wallet (formerly ATB Wallet / DigiByte Mobile)
- **Package:** `app.aroundtheblock.wallet`
- **Vendor:** Around The Block LLC
- **Severity:** Low (Session & Client Transport Hygiene)
- **Affected Versions:** v4.1.0–v4.1.1
- **Patched Version:** v4.1.2
- **Researcher:** Lou ([@LouVan](https://github.com/LouVan))
- **Disclosure Date:** 2026-10-16
- **Release Date:** 2026-10-26

---

## Executive Summary

During an independent security assessment of HoldFast Wallet (v4.1.0–v4.1.1) on Android, an architectural session lifecycle desynchronization was identified in the Hub messaging component.

When a user explicitly triggers a sign-out operation from the Hub interface, local credentials and authentication state were reset, but the active WebSocket connection was not terminated. As a result, the established transport remained active without a valid session context.

Around The Block LLC addressed the issue in HoldFast Wallet v4.1.2 by coupling sign-out actions directly to transport termination, enforcing session validation on outgoing messages, and purging cached session data.

---

## Technical Guarantees & Root Cause Overview

Prior to version 4.1.2:
- **Uncoupled Transport Lifecycle:** Discarding user authentication credentials on logout did not propagate a close signal to the underlying WebSocket transport. The socket connection persisted across state transitions.
- **Missing Session Gating on Transport:** Message dispatch functions verified network transport connectivity but did not enforce local user session presence before transmitting data.

---

## Verification

The behavior was validated within an isolated local test harness using mocked network transports:
1. Upon executing the user sign-out sequence, the network transport retained an active connected state.
2. The transport remained capable of receiving frames and dispatching buffered traffic post-logout.
3. Explicit transport termination confirmed immediate cessation of all duplex transmission.

---

## Remediation & Vendor Guarantees

Around The Block LLC fully remediated this behavior in **HoldFast Wallet v4.1.2**. The updated release establishes three specific security guarantees:

1. **Deterministic Transport Closure:** A Hub logout action now explicitly closes the live WebSocket connection immediately.
2. **Session-Bound Transmission:** The client guarantees that no message frames are dispatched without an active, validated user session.
3. **Session Cache Sanitization:** Cached chat history and temporary channel state are wiped upon sign-out.

Official release announcement and acknowledgment:
- GitHub Release: [HoldFast Wallet 4.1.2](https://github.com/JohnnyLawDGB/aroundtheblock-wallet-releases/releases/tag/v4.1.2)
- Google Play Store: HoldFast Wallet (`app.aroundtheblock.wallet`)

---

## Timeline

- **2026-10-16:** Initial vulnerability report submitted to Around The Block security.
- **2026-10-17:** Vendor confirmed receipt, verified report validity, and assigned fix to the upcoming release wave.
- **2026-10-26:** Vendor released HoldFast Wallet v4.1.2 addressing the vulnerability and credited researcher "Lou" in official release notes.
- **2026-10-26:** Public advisory published in compliance with vendor disclosure terms.
