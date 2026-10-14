<img width="807" height="598" alt="Screenshot 2026-10-15 030436" src="https://github.com/user-attachments/assets/a26418ea-fb03-4316-8508-01938ec3de1f" /># 🛡️ Detection and Analysis of Brute Force Authentication Attacks using Wazuh SIEM

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
<img width="838" height="302" alt="Screenshot 2026-10-04 215803" src="https://github.com/user-attachments/assets/3c023f02-5a4c-4e13-8f5f-bb222320a830" />

---
## 4. Attack Simulation
To test the SIEM’s real-time alert capabilities, a high-velocity authentication brute-force attempt was executed directly on the Windows 11 endpoint using local process invocation (net use local IPC share) :
<img width="700" height="140" alt="Screenshot 2026-10-04 224049" src="https://github.com/user-attachments/assets/72de0aab-0e1b-4825-80ab-76801f707ec5" />

Attack Parameters : 
* Target account : Administrator 
* Attack vector : Network Connection Request (Logon Type 3)
* Velocity & pattern : 15 request executed within 3 seconds to mimic automated credential spraying/ brute-forcing

---
## 5. Telemetry & SIEM Alert Investigation
To test the SIEM’s real-time alert capabilities, a high-velocity authentication brute-force attempt was executed directly on the Windows 11 endpoint using local process invocation (net use local IPC share) :
<img width="881" height="320" alt="Screenshot 2026-10-15 025553" src="https://github.com/user-attachments/assets/77d424dc-807c-45dd-a034-27c7f1d9e99b" />
> **Figure 5.1:** *Wazuh SIEM Dashboard Overview during Attack Simulation.*

<img width="926" height="667" alt="Screenshot 2026-10-05 204604" src="https://github.com/user-attachments/assets/f3a3424e-0e0f-45b2-8c40-aafc6ab397cd" />
> **Figure 5.2:** *Granular Threat Hunting Logs & Event Correlation.*

<img width="807" height="598" alt="Screenshot 2026-10-15 030436" src="https://github.com/user-attachments/assets/e95ce939-ed46-4c89-86e0-6fe28549c322" />


| Telemetry Field | Logged Value | SOC Analyst Analysis |
| :--- | :--- | :--- |
| `agent.name` | Windows-VM-Agent | Identifies the targeted host endpoint within the lab environment |
| `data.win.system.eventID` | 4625 | Standard Windows Security Event ID : An account failed to log on |
| `rule.id` | 60122 | Wazuh Core Detection Rule: Logon Failure- Unknown user or bad password |
| `rule.level` | 5 | Alert Severity : Medium level notification |
| `data.win.eventdata.targetUserName` | Administrator | Indicates targeting of a default privileged account |
| `data.win.eventdata.logonType` | 3 | Network Logon (SMB/ Shared Folder / Remote Network connection) |
---
## 6. Incident Analysis & Key Findings
1. 100% Detection Rate : Wazuh successfully captured all simulated failure attempts 
2. Pattern Recognition : High-density clustering of Event ID 4625 with identical targetUserName : Administrator and LogonType : 3 within seconds confirms automated credential-guessing behavior rather than human password error
3. Risk Exposure : Targeting default privileged accounts poses a severe risk if password lockout policies are missing or weak

---
## 7. Recommended Response & Mitigation Strategy
1.**Enforce Account Lockout Policy**: Configure Group Policy (GPO) to lock accounts after 5 failed attempts within a 5-minute window.
2. **Account Hardening**: Rename or disable the default Administrator account to render automated scripts ineffective.
2. **Automated Incident Response (Active Response)**: Configure Wazuh Active Response scripts to automatically ban or block source IP addresses via Windows Defender Firewall upon triggering Rule 60122 multiple times.

---
## 8. Lessons Learned & Skills Demonstrated
* **Security Operations & SIEM**: Log ingestion, event correlation, and rule parsing using Wazuh.
* **Endpoint Telemetry**: Deep understanding of Windows Security Event IDs (4625) and Logon Types (LogonType 3).
* **Threat Simulation**: Scripting adversary techniques in PowerShell to test monitoring capabilities.
* **Documentation & Reporting**: Structure incident findings into actionable SOC investigation reports.

