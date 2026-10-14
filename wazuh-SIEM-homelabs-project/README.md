# 🛡️ Detection and Analysis of Brute Force Authentication Attacks using Wazuh SIEM

![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-blue?style=for-the-badge&logo=wazuh)
![Windows 11](https://img.shields.io/badge/Target-Windows%2011-0078D4?style=for-the-badge&logo=windows)
![Ubuntu](https://img.shields.io/badge/Manager-Ubuntu%20Linux-E95420?style=for-the-badge&logo=ubuntu)
![PowerShell](https://img.shields.io/badge/Simulation-PowerShell-5391FE?style=for-the-badge&logo=powershell)

---

## 1.  Project Title & Overview
* **Project Title:** End-to-End Threat Detection and Telemetry Analysis of Brute Force Attacks
* **Objective:** To build a controlled Virtual SOC environment using Wazuh SIEM, simulate an automated authentication brute-force attack on a Windows 11 endpoint, analyze security log telemetry, and propose actionable mitigation strategies.

---

## 2.  Environment & Architecture

### Attack & Telemetry Flow:
```text
[1. PowerShell Script] ──> Generates high-velocity failed login attempts on Windows 11
                                   │
                                   ▼
[2. Windows Event Log] ──> Captures Security Event ID 4625 (Logon Failure)
                                   │
                                   ▼
[3. Wazuh Agent]       ──> Collects event log and forwards telemetry to Ubuntu Manager
                                   │
                                   ▼
[4. Wazuh Manager]     ──> Parses log, correlates events, and triggers Rule 60122 (Level 5)
                                   │
                                   ▼
[5. Wazuh Dashboard]   ──> Displays spike (17+ hits) in real-time Threat Hunting dashboard
```
---
## 3. Pre-Simulation Setup & Hardening Configuration 
Windows Audit Policy Enforcement
Enabled login failure auditing via Administrator Powershell to capture failed  authentication events:
![Descriptive Text]("C:\Users\acer\Pictures\Screenshots\Screenshot 2026-10-04 215803.png")

---
## 4. Attack Simulation
To test the SIEM’s real-time alert capabilities, a high-velocity authentication brute-force attempt was executed directly on the Windows 11 endpoint using local process invocation (net use local IPC share) :
![Attack Simulation](C:\Users\acer\Pictures\Screenshots\Screenshot 2026-10-04 224049.png)

* **Attack Parameters : 
Target account : Administrator 
Attack vector : Network Connection Request (Logon Type 3)
Velocity & pattern : 15 request executed within 3 seconds to mimic automated credential spraying/ brute-forcing

