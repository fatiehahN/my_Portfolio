
# 🛡️ Automated Threat Detection, VirusTotal Hash Analysis, and Active Response Mitigation using Wazuh SIEM 


![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-blue?style=for-the-badge&logo=wazuh)
![Windows 11](https://img.shields.io/badge/Target-Windows%2011-0078D4?style=for-the-badge&logo=windows)
![Ubuntu](https://img.shields.io/badge/Manager-Ubuntu%20Linux-E95420?style=for-the-badge&logo=ubuntu)
![PowerShell](https://img.shields.io/badge/Simulation-PowerShell-5391FE?style=for-the-badge&logo=powershell)

---

## 1.  Project Title & Overview
* **Project Title:** Automated Threat Detection, VirusTotal Hash Analysis, and Active Response Mitigation using Wazuh SIEM 
* **Objective:** To build and automated endpoint threat detection and remediation pipeline by monitoring real-time directory changes on a Windows endpoint, enriching suspicious file hashes via VirusTotal API, and executing automated active response scripts to neutralize malware in real time.

---

## 2.  Environment & Architecture

### Threat Detection & Active Response Architecture Flow:
```text
[1. File Drop]         ──> Suspicious/malicious file (EICAR) dropped into monitored path 'C:\malware'
                                         │
                                         ▼
[2. FIM Capture]       ──> Wazuh Agent captures file creation via Syscheck (SQLite fim.db)
                                         │
                                         ▼
[3. VT API Query]      ──> Ubuntu Manager receives FIM alert (Rule 554), extracts hashes & queries VirusTotal API
                                         │
                                         ▼
[4. Threat Intel]      ──> VirusTotal returns reputation feedback confirming malicious signature
                                         │
                                         ▼
[5. High-Severity]     ──> Wazuh Manager fires High-Severity Threat Intelligence Alert (Rule 87105)
                                         │
                                         ▼
[6. Active Response]   ──> Automated execution of 'remove-threat.cmd' on Windows endpoint to purge file
```
---
## 3. Configuration & Hardening Setup
1.Target Path Isolation : Created  (C:/malware) on the Windows host to isolate testing payloads from operational binaries

<img width="550" height="252" alt="Screenshot 2026-10-08 211240" src="https://github.com/user-attachments/assets/ebff6272-dffe-475c-9274-1b553cd0885d" />


2.Central Group Configuration : Updated the central agent.conf via the Wazuh Dashboard (Management → Groups → default) to enable real-time tracking across Windows endpoints:

<img width="925" height="363" alt="Screenshot 2026-10-08 211447" src="https://github.com/user-attachments/assets/e5e24b1f-0567-472f-bc0d-2becbd1bc848" />

<img width="581" height="282" alt="Screenshot 2026-10-08 211711" src="https://github.com/user-attachments/assets/a5c95986-6fcc-4015-9ffa-5b02ee9ee5b5" />
* Agent Synchronization : restarted the Windows Wazuh AGENT Service (net stop wazuh && net start wazuh) to sync central XML policies and initialize real-time directory handles in queue\fim\db\fim.db 

---
## 4. Enable VirusTotal Threat Intelligence Integration
Configured the central integrator module on the Ubuntu Wazuh Manager (/var/ossec/etc/ossec/conf) to automate CTI hash enrichment whenever FIM records a file change :

<img width="876" height="146" alt="Untitled design (4)" src="https://github.com/user-attachments/assets/55e6fad6-7d9c-480d-86e4-247e17c1d832" />

---
## 5. Deploy Automated Remediation Script (Active Response)

Because Windows endpoints cannot execute native Linux .sh scripts, a custom WIndows Batch file was deployed to parse incoming alert JSON payloads from stdin and execute unprompted file deletion.

* Deployment Path : 
  C:\Program Files (x86)\ossec-agent\active-response\bin\remove-threat.cmd 
* Why: When Wazuh fires an Active Response event, it passes raw event JSON directly into stdin. This script parses out .data.virustotal.source.file and executes an unprompted file deletion (del /f /q).

<img width="840" height="502" alt="Screenshot 2026-10-08 214710" src="https://github.com/user-attachments/assets/6d4d412f-9123-4b47-8e57-dd74f92d56ba" />

---
## 6. Configure Manager Active Response Triggers
Defined the Active Response execution bindings inside the Ubuntu Manager ossec.conf to fire remove-threat.cmd whenever VirusTotal returns a positive threat detection (Rule 87105): 

<img width="786" height="455" alt="Screenshot 2026-10-15 114701" src="https://github.com/user-attachments/assets/50512973-2788-453e-bef6-eb1a06118842" />

---
## 7. Attack Simulation & Live Verification 
1. Payload Generation : Downloaded the standardized EICAR test string into monitored path via Powershell
   
   <img width="837" height="52" alt="Screenshot 2026-10-09 004808" src="https://github.com/user-attachments/assets/2d58a173-ba46-4e8c-bf80-c1877dd31525" />

2. Behavioral Observations:
* The file eicar.com hit disk at C:\malware\eicar.com.
* Within 2 to 5 seconds, the Active Response script parsed the VT alert and executed forced
  deletion (del /f /q).
* Verification via Test-Path "C:\malware\eicar.com" returned False.


---
## 8.Telemetry & SIEM Alert Investigation

<img width="1917" height="762" alt="Screenshot 2026-10-09 004605" src="https://github.com/user-attachments/assets/0d6fce8d-0b53-4241-955a-3366d580814b" />

> **Figure 8.1** *Wazuh Threat Hunting Dashboard Overview*

<img width="926" height="471" alt="Screenshot 2026-10-09 004455" src="https://github.com/user-attachments/assets/41596530-3c5c-4955-bf61-db2223a2ad2f" />

> **Figure 8.2** *VirusTotal Rule 87105 Alert Trigger*

Filter query rule.id: 87105 triggered on host Windows-VM-Agent at 17:26:44. The rule fired with Severity Level 12, confirming that 66 engines flagged the file c:\malware\eicar.com. 

<img width="1920" height="1080" alt="Untitled design (5)" src="https://github.com/user-attachments/assets/e0142ad8-af5d-4960-b869-c50bdf1b1969" />

> **Figure 8.3** *Granular Event Document & Field Mapping*


| Telemetry Field | Logged Value | SOC Analyst Significance |
| :--- | :--- | :--- |
| `agent.name` | `Windows-VM-Agent` | Identifies the target host endpoint |
| `agent.ip` | `192.168.137.131` | Endpoint network location |
| `data.integration` | `virustotal` | Confirms automated CTI execution module |
| `data.virustotal.found` | `1` | Hash query match confirmed |
| `data.virustotal.positives` | `66` | 66 out of 68 security engines flagged the file |
| `data.virustotal.source.file` | `c:\malware\eicar.com` | Exact target filepath extracted by Active Response |
| `data.virustotal.source.md5` | `44d88612fea8a8f36de82e1278abb02f` | MD5 hash sent to CTI database |
| `rule.id` | `87105` | High Severity VirusTotal Alert Rule |
| `rule.level` | `12` | High Severity Alert (Triggers Active Response) |
| `rule.mitre.id` | `T1203` | MITRE ATT&CK Execution: Exploitation for Client Execution |


<img width="892" height="772" alt="Screenshot 2026-10-09 004713" src="https://github.com/user-attachments/assets/7f3c424f-88dc-4308-a149-f0b7c79aae14" />

> **Figure 8.4** *VirusTotal Engine Intelligence Verification*
Direct verification on VirusTotal confirmed a 66/68 Community Detection Score, classifying the payload under popular threat label virus.eicar/test. 

---
## 9.Live Attack Simulation & Real-Time Remediation
To test the automated detection and response pipeline, a live simulation was executed on the Windows 11 endpoint while monitoring the Wazuh SIEM dashboard.

**Step-by-Step Simulation Flow:**
1. Payload Download: Opened PowerShell as Administrator on the Windows VM and downloaded the standardized EICAR test file into the monitored folder:
   (lInvoke-WebRequest -Uri "https://secure.eicar.org/eicar.com" -OutFile "C:\malware\eicar.com")
2. File Creation: The file eicar instantly appeared inside the C:\malware directory in Windows File Explorer.
3. SIEM Detection & Enrichment: Within seconds, the Wazuh Agent detected the file change and sent the event to the Wazuh Manager, which queried VirusTotal and generated a Level 12 High Severity Alert.
4.  Automated Containment: The Active Response script (remove-threat.cmd) automatically triggered on the endpoint and deleted eicar.com from C:\malware without any manual user intervention.

   <img width="1698" height="797" alt="Screenshot 2026-10-15 143642" src="https://github.com/user-attachments/assets/c490b3c9-d83d-4b2d-9252-8dc8187005e1" />


