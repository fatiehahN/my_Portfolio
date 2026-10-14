
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


**Real-Time File Creation Detection (Rule 554):**

Upon creating a new file in the monitored directory, the Wazuh Syscheck module immediately captured the file creation event from Windows-VM-Agent. The Wazuh Manager correlated the event telemetry and triggered Rule 554 (File added to the system) at Rule Level 5.

<img width="946" height="507" alt="Screenshot 2026-10-06 213413" src="https://github.com/user-attachments/assets/fae0b524-9250-49fd-b335-17e74c625efd" />


**File Creation (Added) Telemetry**

<img width="3360" height="1890" alt="Untitled design (1)" src="https://github.com/user-attachments/assets/3d175891-1cdc-4f5d-94b1-2468d6377e09" />

**File Creation Telemetry Table**

| Telemetry Field | Logged Value | SOC Analyst Interpretation |
| :--- | :--- | :--- |
| `syscheck.path` | `c:\criticaldata\confidential_passwords.txt` | Target path where the new file artefact was created. |
| `syscheck.event` | `added` | Action type confirming a new file system entry. |
| `rule.id` (Level) | `554 (Level 5)` | Wazuh's default alert code for "New File Created" with a Medium severity level. |
| `syscheck.audit.user.name` | `fnbiyy` | Who did it: Proves that the user account fnbiyy physically created this file. |
| `syscheck.uname.after` | `Administrator` | Shows that the file was created under Administrator permission settings. |
| `syscheck.mode` | `whodata` | Confirms real-time tracking is active using Windows Security logs. |
| `syscheck.sha256_after` | `343fcb40497549085c98ae137c137116a5c2442eb8dc0bf0cac3a3419ce05b9f` | Initial SHA-256 cryptographic hash of the new file content. |

### 2. Phase 2 : File Modification (Data Tampering)

Add-Content -Path "C:\CriticalData\confidential_passwords.txt" -Value "`nNew_Admin_Hash_2026"
<img width="848" height="246" alt="Screenshot 2026-10-06 214019" src="https://github.com/user-attachments/assets/0b67c8f5-ecde-4626-b227-4b32ada30a9c" />

**Real-Time File Modified Detection (Rule 550):**

Upon modifying the contents of a file in the monitored directory, the Wazuh Syscheck module instantly detected the content alteration on Windows-VM-Agent. The Wazuh Manager correlated the event telemetry and triggered Rule 550 (Integrity checksum changed.) at Rule Level 7.
<img width="937" height="570" alt="Screenshot 2026-10-06 214302" src="https://github.com/user-attachments/assets/bfbb3ef6-fc2c-4a37-9039-791f9e779015" />

**File Modification Telemetry**

<img width="3360" height="1890" alt="Untitled design (2)" src="https://github.com/user-attachments/assets/80808c15-8bb1-4bd6-92a0-8a12a59ccf5c" />

**File Modification Telemetry Table**

| Telemetry Field | Logged Value | SOC Analyst Interpretation |
| :--- | :--- | :--- |
| `syscheck.path` | `c:\criticaldata\confidential_passwords.txt` | Shows which specific file was edited or tampered with. |
| `syscheck.event` | `modified` | Confirms that someone opened the file and changed its contents. |
| `rule.id` (Level) | `550 (Level 7)` | Wazuh's alert code for "File Content Changed" with a Higher severity level. |
| `syscheck.audit.user.name` | `fnbiyy` | Who did it: Proves that user fnbiyy edited the text inside the file. |
| `syscheck.uname.after` | `Administrator` | Shows the file owner context during the editing process. |
| `syscheck.mode` | `whodata` | Confirms real-time tracking is active using Windows Security logs. |
| `syscheck.sha256_before` | `343fcb40497549085c98ae137c137116a5c2442eb8dc0bf0cac3a3419ce05b9f` | The original file fingerprint before any changes were made. |
| `syscheck.sha256_after` | `ab02902ee43d0a7243e589787c6f56ce232b3ce44b02f197376d2733f5044387` | The new file fingerprint after editing. The difference between before and after proves mathematically that the content was changed. |


### 3. Phase 3 : File Deletion

Remove-Item -Path "C:\CriticalData\confidential_passwords.txt"

<img width="797" height="97" alt="Screenshot 2026-10-06 214447" src="https://github.com/user-attachments/assets/0cf6bf3f-2262-4abb-9a36-8e4dda2141ba" />

**Real-Time File Deletion Detection (Rule 553):**
Upon deleting a file from the monitored directory, the Wazuh Syscheck module instantly captured the deletion event on Windows-VM-Agent. The Wazuh Manager correlated the event telemetry and triggered Rule 553 (File deleted.) at Rule Level 7.

<img width="933" height="438" alt="Screenshot 2026-10-06 214532" src="https://github.com/user-attachments/assets/4eddcb74-53d4-4ec3-8bd1-cad724c3580a" />

**File Deletion  Telemetry**

<img width="3360" height="1890" alt="Untitled design (3)" src="https://github.com/user-attachments/assets/70dcc43a-5854-4a5c-83d4-fcffc86a2528" />

**File Deleted Telemetry Table**

| Telemetry Field | Logged Value | SOC Analyst Interpretation |
| :--- | :--- | :--- |
| `syscheck.path` | `c:\criticaldata\confidential_passwords.txt` | Shows the file path that was removed or destroyed. |
| `syscheck.event` | `deleted` | Confirms that the file was deleted from the hard drive. |
| `rule.id` (Level) | `553 (Level 7)` | Wazuh's alert code for "File Deleted" with a High severity level. |
| `syscheck.audit.user.name` | `fnbiyy` | Who did it: Proves that user fnbiyy deleted the file. |
| `syscheck.uname.after` | `Administrator` | Shows the file owner context during the deletion. |
| `syscheck.mode` | `whodata` | Confirms real-time tracking is active using Windows Security logs. |
| `syscheck.audit.process.name` | `C:\...\powershell.exe` | Tool used: Shows PowerShell was the process used to delete the file. |
| `syscheck.sha256_after` | `ab02902ee43d0a7243e589787c6f56ce232b3ce44b02f197376d2733f5044387` | The last known SHA-256 fingerprint of the file contents right before it was erased. |

---
## 5. Key Finding & SOC Investigation Insights

* **Knowing Who Did It (User Accountability):**
By turning on whodata="yes", Wazuh clearly captured the exact username (syscheck.audit.user.name: fnbiyy). This gives SOC analysts direct proof of WHO created, changed, or deleted the file during an investigation.
* **Mathematical Proof of File Changes (Hash Changes):**
When the file was edited (Rule 550), the SHA-256 fingerprint changed from 343fcb40... (before) to ab02902e... (after). Because two different files cannot have the exact same hash, this difference proves 100% that the file contents were modified.
* **Detecting Ransomware Patterns:**
The quick sequence of Rule 554 (File Created)  Rule 550 (File Edited)  Rule 553 (File Deleted) mimics how real ransomware works: it creates a payload, encrypts your data, and deletes the original file.

---
## 6. Recommended Response & Mitigation Strategy
* **Lock Down Important Folders (Access Controls):**
Restrict folder permissions so that normal users cannot easily edit or delete sensitive files in C:\CriticalData.
* **Raise Alert Levels for System Files:**
Set up custom Wazuh rules to trigger high-severity alerts (Level 12+) if anyone touches critical system folders like C:\Windows\System32\.
* **Automate Threat Response (Active Response):**
Configure Wazuh to automatically disconnect a computer from the network if it detects mass file deletions (Rule 553) in a short time (e.g., 50 files deleted in 10 seconds).


