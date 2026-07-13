# 04 — Network Analysis

![Platform](https://img.shields.io/badge/platform-Android-3DDC84?logo=android&logoColor=white)
![Methodology](https://img.shields.io/badge/methodology-OWASP%20MASVS%20%2F%20MASTG-blue)
![Target](https://img.shields.io/badge/target-InsecureBankv2-critical)
![Phase](https://img.shields.io/badge/phase-Network%20Analysis-orange)

> Network-layer analysis of InsecureBankv2 — intercepting, inspecting, and manipulating traffic between the application and its backend to assess transport security, authentication handling, and server-side trust.

This section documents the network analysis phase of the broader mobile application reverse engineering assessment. It sits alongside `03-Dynamic-Analysis`, focusing specifically on what crosses the wire rather than what happens inside the process.

---

## Objective

Assess the confidentiality and integrity of data in transit by placing the application in a controlled man-in-the-middle position — evaluating transport encryption, certificate validation, and the security of API endpoints and session handling.

## Scope

| Item | Detail |
|---|---|
| Target application | InsecureBankv2 |
| Analysis type | Network traffic interception & manipulation |
| Environment | Rooted emulator/device routed through an intercepting proxy |
| Methodology | OWASP MASVS & MASTG |

## Toolchain

| Tool | Purpose |
|---|---|
| **Burp Suite** | Intercepting proxy — traffic capture, request/response tampering, repeater/intruder testing |
| **ADB** | Proxy configuration, certificate installation, device-side traffic setup |
| **Frida / Objection** | SSL pinning bypass to enable interception where pinning is present |
| **MobSF** | Automated network security triage (cleartext traffic, weak TLS config detection) |

## Key Findings (Summary)

| # | Finding | MASVS Category | Severity |
|---|---|---|---|
| 1 | Cleartext HTTP traffic — credentials/session data transmitted unencrypted | MASVS-NETWORK | Critical |
| 2 | No certificate/SSL pinning implemented | MASVS-NETWORK | High |
| 3 | Sensitive parameters exposed in URL/request bodies | MASVS-NETWORK | Medium |
| 4 | Weak or absent server-side session/token validation observed in transit | MASVS-AUTH | High |

> Full technical detail — captured requests, proxy history, and remediation guidance — is covered in the main research report; this README is a high-level index for the repository.

## Methodology Approach

1. **Proxy setup** — configure device/emulator to route all traffic through Burp Suite; install Burp's CA certificate on-device.
2. **Baseline capture** — observe normal application traffic to map endpoints, headers, and data flows.
3. **Transport review** — check for cleartext HTTP usage, mixed content, and TLS configuration weaknesses.
4. **Pinning bypass (where needed)** — use Frida/Objection to defeat certificate pinning and restore visibility into encrypted traffic.
5. **Request tampering** — use Repeater/Intruder to test server-side trust in client-supplied data (parameter tampering, replay, session handling).

## Repository Contents

```
04-Network-Analysis/
├── Assets/        # Evidence: Burp proxy captures, request/response screenshots
└── README.md      # This file
```

## MITRE ATT&CK Mobile Mapping

Findings from this phase map primarily to collection and credential access techniques — unencrypted transport and absent pinning enable network-position adversaries to intercept in-transit credentials and session tokens. See the main report for the full mapping table.

## Related Sections

- `02-Static-Analysis` — Decompilation and source-level code review
- `03-Dynamic-Analysis` — Runtime instrumentation (Frida, Objection)

---
*Part of the [Reverse-Engineering-In-Mobile-Application](../) assessment repository.*
