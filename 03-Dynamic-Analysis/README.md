# 03 — Dynamic Analysis

![Platform](https://img.shields.io/badge/platform-Android-3DDC84?logo=android&logoColor=white)
![Methodology](https://img.shields.io/badge/methodology-OWASP%20MASVS%20%2F%20MASTG-blue)
![Target](https://img.shields.io/badge/target-InsecureBankv2-critical)
![Phase](https://img.shields.io/badge/phase-Dynamic%20Analysis-orange)

> Dynamic (DAST) analysis of InsecureBankv2 — instrumenting and observing the application **at runtime** on a live device/emulator to validate, exploit, and expand on the flaws identified during static analysis.

This section documents the runtime instrumentation phase of the broader mobile application reverse engineering assessment. It builds directly on `02-Static-Analysis`, using live execution to confirm exploitability, intercept traffic, and bypass in-app protections that source review alone cannot fully verify.

---

## Objective

Validate and exploit vulnerabilities under real execution conditions — intercepting live network traffic, hooking runtime methods to bypass client-side controls, and inspecting the application's in-memory and on-device behavior — to demonstrate real-world impact beyond static findings.

## Scope

| Item | Detail |
|---|---|
| Target application | InsecureBankv2 |
| Analysis type | Dynamic / runtime instrumentation, black-box + grey-box |
| Environment | Rooted Android emulator/device |
| Methodology | OWASP MASVS & MASTG |

## Toolchain

| Tool | Purpose |
|---|---|
| **Frida** | Runtime instrumentation, function hooking, bypassing client-side checks |
| **Objection** | Frida-powered runtime exploration (SSL pinning bypass, root detection bypass, storage inspection) |
| **Burp Suite** | Intercepting proxy for inspecting/manipulating HTTP(S) traffic |
| **ADB** | Device interaction, log capture, app install/data access |
| **MobSF** | Dynamic analysis mode for automated runtime triage and cross-verification |

## Key Findings (Summary)

| # | Finding | MASVS Category | Severity |
|---|---|---|---|
| 1 | Lack of SSL/TLS pinning — traffic interceptable via proxy | MASVS-NETWORK | High |
| 2 | Runtime bypass of root/tamper detection | MASVS-RESILIENCE | Medium |
| 3 | Sensitive data exposed in runtime memory / logs | MASVS-STORAGE | High |
| 4 | Business logic flaws confirmed via live request tampering | MASVS-AUTH | Critical |

> Full technical detail — Frida scripts, intercepted request/response pairs, and remediation guidance — is covered in the main research report; this README is a high-level index for the repository.

## Methodology Approach

1. **Environment setup** — rooted emulator, Burp CA cert installed, Frida server deployed to device.
2. **Traffic interception** — route app traffic through Burp Suite to inspect and tamper with live requests.
3. **Runtime hooking** — use Frida/Objection to bypass SSL pinning, root detection, and other client-side guardrails.
4. **Storage & memory inspection** — use Objection to dump SharedPreferences, SQLite, and in-memory data during active sessions.
5. **Automated cross-check** — MobSF dynamic scan to validate manual findings and surface additional behavioral flags.

## Repository Contents

```
03-Dynamic-Analysis/
├── Assets/        # Evidence: Burp captures, Frida hook output, runtime screenshots
└── README.md      # This file
```

## MITRE ATT&CK Mobile Mapping

Findings from this phase map to defense evasion and collection techniques (e.g. SSL pinning bypass → `T1509` weakened credential/data protections; root detection bypass → evasion of mobile-specific detection mechanisms). See the main report for the full mapping table.

## Related Sections

- `02-Static-Analysis` — Decompilation and source-level code review
- `04-*` — Reporting / Remediation (if applicable)

---
*Part of the [Reverse-Engineering-In-Mobile-Application](../) assessment repository.*
