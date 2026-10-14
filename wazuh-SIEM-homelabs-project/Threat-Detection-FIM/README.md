
# 🛡️ Real-Time Endpoint File Integrity Monitoring & Threat Detection via Wazuh FIM Module


![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-blue?style=for-the-badge&logo=wazuh)
![Windows 11](https://img.shields.io/badge/Target-Windows%2011-0078D4?style=for-the-badge&logo=windows)
![Ubuntu](https://img.shields.io/badge/Manager-Ubuntu%20Linux-E95420?style=for-the-badge&logo=ubuntu)
![PowerShell](https://img.shields.io/badge/Simulation-PowerShell-5391FE?style=for-the-badge&logo=powershell)

---

## 1.  Project Title & Overview
* **Project Title:** Real-Time Endpoint File Integrity Monitoring (FIM) and Data Tempering Detection
* **Objective:** Deploy and configure the Wazuh SIEM Syscheck module to perform real-time File Integrity Monitoring (FIM) on a Windows 11 endpoint. The goal is to detect unauthorized file creations, content modifications, and deletions within sensitive directories, capturing forensic telemetry (whoadata attribution and cryptographic hash changes) to simulate anti-ransomware and insider threat monitoring workflows.

---

## 2.  Environment & Architecture

### Telemetry & Monitoring Architecture Flow:
```text
[1. Target Actions]     ──> PowerShell generates file Creation, Modification, & Deletion in 'C:\CriticalData'
                                           │
                                           ▼
[2. SACL & Syscheck]    ──> Windows SACL & Wazuh Agent capture file events & user attribution ('whodata')
                                           │
                                           ▼
[3. Agent Telemetry]    ──> Packages event telemetry (Paths, Processes, & SHA-256 Hashes) over Ports 1514/1515
                                           │
                                           ▼
[4. Wazuh Manager]      ──> Correlates events & triggers Rule 554 (Added), Rule 550 (Modified), or Rule 553 (Deleted)
                                           │
                                           ▼
[5. Wazuh Dashboard]    ──> Visualizes real-time FIM alerts, user attribution, and hash differentials
```
---
## 3. Configuration & Hardening Setup
1. Endpoint Directory Provisioning
Created a designated high-value target directory via PowerShell (Admin):

<img width="588" height="160" alt="Screenshot 2026-10-06 205713" src="https://github.com/user-attachments/assets/a7349922-7235-4142-8a30-30ae4a2dea60" />

2. Wazuh Agent FIM Engine Tuning (ossec.conf)
Configured the <syscheck> block inside C:\Program Files (x86)\ossec-agent\ossec.conf to enable real-time detection, hash delta reporting, and user account attribution (whodata):

<img width="1830" height="510" alt="Untitled design" src="https://github.com/user-attachments/assets/d6a01189-e54b-4210-869a-2032487c0607" />

3. Service Synchronization
Restarted the local agent service to push new monitoring directives to the Wazuh Manager:

---
## 4. Threat Simulation
Three distinct file system manipulation lifecycle events were simulated sequentially using PowerShell to evaluate SIEM detection coverage: 

### 1. Phase 1 : File Creation 
New-Item -Path "C:\CriticalData\confidential_passwords.txt" -ItemType File -Value "SuperSecret123"

<img width="842" height="175" alt="Screenshot 2026-10-06 212238" src="https://github.com/user-attachments/assets/767f2517-3385-431a-96cc-7935cd4d9613" />


Real-Time File Creation Detection (Rule 554):
Upon creating a new file in the monitored directory, the Wazuh Syscheck module immediately captured the file creation event from Windows-VM-Agent. The Wazuh Manager correlated the event telemetry and triggered Rule 554 (File added to the system) at Rule Level 5.

<img width="946" height="507" alt="Screenshot 2026-10-06 213413" src="https://github.com/user-attachments/assets/fae0b524-9250-49fd-b335-17e74c625efd" />


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

