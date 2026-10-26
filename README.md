# Security Advisories & Vulnerability Research

Independent security research, vulnerability disclosures, and technical write-ups by **Lou** ([@LouVan](https://github.com/LouVan)).

This repository documents responsible disclosures, bug bounty findings, and architectural security analyses across mobile clients, Web3/DeFi ecosystems, and decentralized identity protocols.

---

## Published Advisories

| Advisory ID | Target / Ecosystem | Vulnerability Title | Severity | Status | Release / Fix |
|---|---|---|:---:|:---:|:---:|
| [**ATB-2026-01**](./advisories/ATB-2026-01-holdfast-hub-logout.md) | DigiByte / HoldFast Wallet (`app.aroundtheblock.wallet`) | Hub WebSocket Session Desynchronization & Non-Termination on Logout | **Low** | Resolved | [HoldFast v4.1.2](https://github.com/JohnnyLawDGB/aroundtheblock-wallet-releases/releases/tag/v4.1.2) |

---

## Research Focus Areas

- **Mobile Client Security:** Android native reverse engineering, JNI bridge analysis, IPC boundaries, session lifecycle state machines.
- **Web3 & Wallet Architecture:** SPV consensus assumptions, UTXO/asset attribution logic, cross-protocol state synchronization.
- **Protocol & API Security:** Authorization bypasses, BOLA/IDOR, session non-invalidation, logic flaws.

---

## Responsible Disclosure & Policy Compliance

All research is conducted in full compliance with project security policies and coordinated vulnerability disclosure (CVD) timelines:
- Vendors are notified privately with reproducible test cases prior to publication.
- Technical write-ups focus strictly on architectural root causes, impact analysis, and verified remediation guarantees.
- Weaponized exploit code and malicious payload templates are omitted to protect users.
