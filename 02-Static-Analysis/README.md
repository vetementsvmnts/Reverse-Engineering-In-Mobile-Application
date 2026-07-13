# 02 — Static Analysis

![Platform](https://img.shields.io/badge/platform-Android-3DDC84?logo=android&logoColor=white)
![Methodology](https://img.shields.io/badge/methodology-OWASP%20MASVS%20%2F%20MASTG-blue)
![Target](https://img.shields.io/badge/target-InsecureBankv2-critical)
![Phase](https://img.shields.io/badge/phase-Static%20Analysis-orange)

> Static (SAST) analysis of the InsecureBankv2 APK — decompiling, disassembling, and manually reviewing the application's code, resources, and configuration **without execution**, to identify vulnerabilities baked into the binary itself.

This section documents the static analysis phase of the broader mobile application reverse engineering assessment. It picks up after initial reconnaissance and precedes dynamic/runtime analysis, forming the "white-box code review" leg of the assessment.

---

## Objective

Identify design and implementation flaws visible at rest in the APK — manifest misconfigurations, insecure cryptography, unsafe data storage, and hardcoded secrets — by decompiling the application and reviewing source, bytecode, and resources directly.

## Scope

| Item | Detail |
|---|---|
| Target application | InsecureBankv2 |
| Analysis type | Static / white-box, no runtime instrumentation |
| Package format | Android APK |
| Methodology | OWASP MASVS & MASTG |

## Toolchain

| Tool | Purpose |
|---|---|
| **JADX-GUI** | Java source decompilation & manual code review |
| **apktool** | APK decoding, smali disassembly, resource/manifest extraction |
| **MobSF** | Automated static scan for triage and cross-verification |

## Key Findings (Summary)

| # | Finding | MASVS Category | Severity |
|---|---|---|---|
| 1 | `AndroidManifest.xml` misconfigurations (exported components, `debuggable`/`allowBackup` flags) | MASVS-PLATFORM | High |
| 2 | Hardcoded AES encryption key with a static IV | MASVS-CRYPTO | Critical |
| 3 | Insecure local data storage (plaintext SharedPreferences / SQLite) | MASVS-STORAGE | High |
| 4 | Cleartext HTTP communication | MASVS-NETWORK | High |

> Full technical detail, code excerpts, and remediation guidance for each finding are covered in the main research report; this README is a high-level index for the repository.

## Methodology Approach

1. **Decode & disassemble** — `apktool` to extract manifest, resources, and smali.
2. **Decompile** — `JADX-GUI` to recover readable Java source for manual review.
3. **Manifest review** — audit exported activities/services/receivers, permissions, and security-relevant flags.
4. **Code review** — trace cryptographic routines, storage APIs, and network calls for insecure implementation patterns.
5. **Automated cross-check** — `MobSF` scan to validate manual findings and catch anything missed.

## Repository Contents

```
02-Static-Analysis/
├── Assets/        # Evidence: screenshots, decompiled snippets, manifest excerpts
└── README.md      # This file
```

## MITRE ATT&CK Mobile Mapping

Findings from this phase map to reconnaissance and credential/data access techniques (e.g. insecure data storage → `T1409` Access Stored Application Data; hardcoded crypto material → weakened confidentiality controls exploitable in later phases). See the main report for the full mapping table.

## Related Sections

- `01-*` — Reconnaissance / Environment Setup
- `03-Dynamic-Analysis` — Runtime instrumentation (Frida, Objection, Burp Suite)

---
*Part of the [Reverse-Engineering-In-Mobile-Application](../) assessment repository.*
