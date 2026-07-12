<div align="center">

# Reverse-Engineering-In-Mobile-Application

### Static & Dynamic Bytecode Analysis Framework for Android Application Security

*Deconstructing mobile binaries. Identifying structural vulnerabilities. Designing hardware-backed remediation.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Android-3DDC84?logo=android&logoColor=white)](#)
[![Methodology](https://img.shields.io/badge/Methodology-OWASP%20MASVS%20%2F%20MASTG-blue)](#)
[![Framework](https://img.shields.io/badge/Mapped%20To-MITRE%20ATT%26CK-red)](#)
[![Status](https://img.shields.io/badge/Status-Active%20Research-brightgreen)](#)

</div>

---

## Overview

This repository documents a full-cycle **mobile application security assessment** of a deliberately vulnerable Android banking application (**InsecureBankv2**), combining **Static Application Security Testing (SAST)**, **Dynamic Application Security Testing (DAST)**, and **bytecode-level reverse engineering** to identify, validate, and remediate structural weaknesses across the application's attack surface.

The assessment follows the **OWASP MASVS / MASTG** methodology end-to-end — from manifest analysis and static decompilation through to runtime instrumentation and traffic interception — producing evidence-backed findings mapped to industry frameworks and prioritized by exploitability and business impact.

> **Objective:** Simulate a real-world mobile penetration test at a level suitable for enterprise reporting — not a checklist exercise, but a defensible, reproducible chain of evidence from discovery to remediation.

---

## 🧭 Table of Contents

- [Methodology](#-methodology)
- [Toolchain](#-toolchain)
- [Key Findings](#-key-findings)
- [Risk Matrix](#-risk-matrix)
- [Attack Chain Mapping](#-attack-chain-mapping-mitre-attck)
- [Remediation Architecture](#-remediation-architecture)
- [Repository Structure](#-repository-structure)
- [Reproducing This Assessment](#-reproducing-this-assessment)
- [Disclaimer](#-disclaimer)
- [Author](#-author)

---

## 🧪 Methodology

Assessment structured around the **OWASP Mobile Application Security Testing Guide (MASTG)**, aligned to control groups defined in the **OWASP MASVS**:

| Phase | Focus | Techniques |
|---|---|---|
| **1. Reconnaissance** | APK structure, manifest, permissions | `apktool`, `aapt`, manifest diffing |
| **2. Static Analysis (SAST)** | Decompiled source & bytecode review | JADX-GUI, Smali inspection, string/entropy analysis |
| **3. Dynamic Analysis (DAST)** | Runtime behavior, memory, storage | Frida instrumentation, `ADB` shell inspection |
| **4. Network Analysis** | Transport-layer security | Burp Suite MITM proxy, certificate pinning review |
| **5. Automated Correlation** | Cross-validation of manual findings | MobSF automated scan baseline |
| **6. Reporting** | Risk scoring, remediation design | CVSS-aligned severity, MASVS control mapping |

---

## 🛠️ Toolchain

<div align="center">

| Category | Tools |
|---|---|
| **Decompilation** | `JADX-GUI`, `apktool`, `dex2jar` |
| **Instrumentation** | `Frida`, `Objection` |
| **Interception** | `Burp Suite Professional` |
| **Device Control** | `ADB` (Android Debug Bridge) |
| **Automated Scanning** | `MobSF` (Mobile Security Framework) |
| **Environment** | Android Virtual Device (AVD), rooted emulator |

</div>

---

## 🎯 Key Findings

| # | Finding | MASVS Control | Severity |
|---|---|---|---|
| 1 | **Hardcoded AES encryption key** embedded in application bytecode | MASVS-CRYPTO-1 | 🔴 Critical |
| 2 | **Insecure local data storage** — sensitive data persisted in cleartext (SharedPreferences / SQLite) | MASVS-STORAGE-1 | 🔴 Critical |
| 3 | **Cleartext HTTP transmission** of authentication and session data | MASVS-NETWORK-1 | 🟠 High |
| 4 | **AndroidManifest misconfigurations** — exported activities/services without permission enforcement | MASVS-PLATFORM-1 | 🟠 High |
| 5 | **Absent certificate pinning**, enabling active MITM interception | MASVS-NETWORK-2 | 🟡 Medium |
| 6 | **Debuggable build flag** enabled in production-representative build | MASVS-RESILIENCE-1 | 🟡 Medium |

Each finding includes: root-cause bytecode/manifest evidence, proof-of-concept reproduction steps, and CVSS-aligned scoring — detailed in the full technical report.

---

## 📊 Risk Matrix

```
IMPACT
  │
H │   [4]         [1] [2]
I │                       
G │                        
H │            [5]     [3]
  │
M │                [6]
E │
D │
  │
L │
O │
W │
  └──────────────────────────────
    LOW      MEDIUM      HIGH
              LIKELIHOOD
```

**Legend:** `[1]` Hardcoded AES Key · `[2]` Insecure Storage · `[3]` Cleartext Transmission · `[4]` Manifest Misconfiguration · `[5]` No Cert Pinning · `[6]` Debuggable Build

---

## ⚔️ Attack Chain Mapping (MITRE ATT&CK)

| Tactic | Technique | Application |
|---|---|---|
| **Discovery** | T1420 – File and Directory Discovery | Local storage inspection via ADB shell |
| **Credential Access** | T1409 – Stored Application Data | Extraction of cleartext credentials from local DB |
| **Collection** | T1533 – Data from Local System | Harvesting session tokens from SharedPreferences |
| **Command and Control** | T1437 – Application Layer Protocol | Interception of unencrypted API traffic |
| **Defense Evasion** | T1577 – Compromise Application Executable | Reverse-engineered logic exposing key material |

---

## 🏗️ Remediation Architecture

Proposed hardening model moving cryptographic trust **out of application bytecode** and into hardware-backed isolation:

```
┌─────────────────────────────┐
│      Application Layer      │
│   (no embedded key material)│
└──────────────┬───────────────┘
               │  Key requests only
               ▼
┌─────────────────────────────┐
│   Android Keystore System   │
│   (hardware-backed, TEE)    │
└──────────────┬───────────────┘
               │
               ▼
┌─────────────────────────────┐
│  Trusted Execution Environ. │
│   / StrongBox (if present)  │
└─────────────────────────────┘
```

**Core recommendations:**
- Migrate all cryptographic key material to **Android Keystore** (hardware-backed where available)
- Enforce **TLS 1.2+ with certificate pinning** on all network calls
- Apply **`exported=false`** by default on non-essential components; explicit intent filtering
- Encrypt local storage using **EncryptedSharedPreferences / SQLCipher**
- Strip **`debuggable`** and enable **ProGuard/R8** obfuscation for release builds

---

## 📁 Repository Structure

```
Reverse-Engineering-In-Mobile-Application/
├── README.md
├── LICENSE
├── report/                # Full SAST/DAST technical report
├── evidence/               # Screenshots, decompiled snippets, traffic captures
├── static-analysis/        # JADX output, manifest review, string/entropy findings
├── dynamic-analysis/       # Frida scripts, ADB session logs
└── network-analysis/       # Burp Suite project files, request/response captures
```

---

## ▶️ Reproducing This Assessment

```bash
# 1. Decompile the APK
apktool d insecurebankv2.apk -o output/
jadx-gui insecurebankv2.apk

# 2. Deploy to controlled AVD (rooted emulator)
adb install insecurebankv2.apk
adb shell

# 3. Instrument with Frida
frida -U -f com.android.insecurebankv2 -l hook.js --no-pause

# 4. Intercept traffic via Burp Suite
adb shell settings put global http_proxy <burp_ip>:8080
```

> Full step-by-step reproduction, including Frida hook scripts and Burp configuration, is documented in `report/`.

---

## ⚠️ Disclaimer

This project targets **InsecureBankv2**, an intentionally vulnerable application designed for educational security research. All testing was conducted in an **isolated, controlled lab environment**. Techniques documented here are intended strictly for **authorized security assessment, training, and research purposes**.

---

## 👤 Author

**Kitsana Thuekoh**
Penetration Tester & Vulnerability Researcher · Johannesburg, South Africa

`CPTS` · `OSCP` · `CompTIA PenTest+` · `CompTIA Security+` · `ISC2 CC` · NASA VDP Letter of Recognition

[GitHub](https://github.com/vetementsvmnts) · [LinkedIn](https://linkedin.com/in/kitsana-thuekoh-66ba9b314)

</div>
