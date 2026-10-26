# Advisory ATB-2026-01: Hub WebSocket Session Desynchronization & Non-Termination on Logout

- **Target:** HoldFast Wallet (formerly ATB Wallet / DigiByte Mobile)
- **Package:** `app.aroundtheblock.wallet`
- **Vendor:** Around The Block LLC
- **Severity:** Low (Session & Client Transport Hygiene)
- **Affected Versions:** v4.1.0 (commit `8fe2440ef`)
- **Patched Version:** v4.1.2 (commit `v4.1.2`)
- **Researcher:** Lou ([@LouVan](https://github.com/LouVan))
- **Disclosure Date:** 2026-10-16
- **Release Date:** 2026-10-26

---

## Executive Summary

During an independent security assessment of HoldFast Wallet v4.1.0 on Android, an architectural session lifecycle desynchronization was identified in the Hub social messaging component. 

When a user explicitly triggers a logout operation from the Hub profile screen, the application resets UI state and clears the local authentication token, but fails to terminate the underlying singleton WebSocket transport connection. Because message transmission routines lacked local session state assertion checks, the active socket remained capable of duplex communication without an authenticated user context.

Around The Block LLC remediated the issue in HoldFast Wallet v4.1.2 by coupling the logout lifecycle to the socket connection manager, ensuring immediate socket closure, session gating on outbound traffic, and cache purge.

---

## Technical Analysis & Root Cause

### 1. Transport Lifecycle Desynchronization
In v4.1.0, the Hub chat infrastructure was configured as a singleton dependency injected across the application lifecycle (`AppModule.kt`). 

When an authenticated user navigated to `ProfileViewModel.logout()` to sign out of the Hub, the invocation sequence executed:
```kotlin
// ProfileViewModel.kt
digiScopeClient.logout()
_uiState.update { it.copy(isLoggedIn = false) }
```

Inside `DigiScopeClient.logout()`, the local credential token was discarded and an asynchronous token revocation request was enqueued. However, `HubWebSocket.disconnect()` was never called along this execution path. 

### 2. State Assumption vs. Transport State
The only call sites invoking `HubWebSocket.disconnect()` were:
- `ChatViewModel.onCleared()`
- `IdentitySessionEraser.wipe()` (full wallet reset)

Because the Chat and Profile views share a single navigation host without clearing the ViewModel scope, switching between tabs or logging out left the WebSocket transport in a persistent `CONNECTED` state.

### 3. Missing Guard on Message Dispatch
Inspection of outbound dispatch methods (`sendMessage()`, `sendTyping()`, `joinChannel()`) revealed that the methods verified socket connection state (`state == CONNECTED`) but did not evaluate session token validity before framing and transmitting packets onto the wire.

Consequently:
- The server could continue emitting live inbound chat frames to a client whose user had signed out.
- The client could continue dispatching frames over the pre-established socket despite local token clearance.

---

## Verification & Proof of Concept

The vulnerability was validated within the project's native build system using isolated unit test harnesses driving the production `HubWebSocket` and `ProfileViewModel` classes with mocked network transports:

1. **State Retention:** Upon executing the logout sequence, the socket state remained `CONNECTED`.
2. **Duplex Liveness:** Inbound message frames were received and processed post-logout. Outbound frames (`sendMessage`, `sendTyping`) were successfully dispatched to the mock transport.
3. **Decisive Control:** Explicit invocation of `disconnect()` immediately ceased all transmission activity.

---

## Remediation & Vendor Guarantees

Around The Block LLC addressed the advisory in version **4.1.2**:

1. **Coupled Lifecycle Termination:** The Hub logout routine now explicitly invokes `disconnect()` on the active WebSocket instance.
2. **Session-Bound Gating:** Outbound frame dispatch strictly requires an active, validated session context.
3. **Cache Purge:** Cached chat messages and local channel artifacts are sanitized from volatile memory upon sign-out.

Official release announcement and acknowledgment:
- GitHub Release: [HoldFast Wallet 4.1.2](https://github.com/JohnnyLawDGB/aroundtheblock-wallet-releases/releases/tag/v4.1.2)
- Google Play Store: HoldFast Wallet (`app.aroundtheblock.wallet`)

---

## Timeline

- **2026-10-16:** Initial vulnerability report and technical reproduction submitted to Around The Block security.
- **2026-10-17:** Vendor confirmed receipt, verified report validity, and assigned fix to the upcoming release wave.
- **2026-10-26:** Vendor released HoldFast Wallet v4.1.2 addressing the vulnerability and credited researcher "Lou" in official release notes.
- **2026-10-26:** Public advisory published in compliance with vendor disclosure terms.
