# 🛡️ LetsDefend Lab Write-Up: SOC145 - Ransomware Detected

## 📌 Executive Summary
* **Platform:** LetsDefend
* **Alert Name:** SOC145 - Ransomware Detected
* **Objective:** Investigate a ransomware execution alert on an enterprise endpoint, analyze binary artifacts, check for network communication/C2 activity, and contain the compromised system.

---

## 🔍 Initial Alert Details
* **Event ID  :** 92
* **Event Time:** May 23, 2021, 07:32 PM
*  **Rule :** SOC145 - Ransomware Detected
*  **Alert Type :** Malware
*  **MITRE ATT&CK :** T1204
* **Source IP:** 172.16.17.88
* **Source Hostname:** MarkPRD
* **File Name:** `ab.exe`
* **File Hash (MD5):** `0b486fe0503524cfe4726a4022fa6a68`
* **File Size:** 775.50 KB
* **Device Action:** Allowed

<img width="767" height="372" alt="Screenshot 2026-10-14 200349" src="https://github.com/user-attachments/assets/8a6bd3b1-1b07-4738-a135-d2cd5317863e" />

---

## 📋 Step-by-Step Investigation Process

### Step 1: Initial Alert Review
* Reviewed alert metadata for Event ID 92 (`SOC145 - Ransomware Detected`).
* Observed that the **Device Action** was marked as **Allowed**, indicating that security controls failed to block or isolate the binary upon arrival and execution.

### Step 2: VirusTotal Threat Intelligence Analysis
* Searched the file hash (`0b486fe0503524cfe4726a4022fa6a68`) on **VirusTotal**.
* **Finding:** Flagged by **57 out of 66** security vendors as malicious.
* **Malware Family:** Identified as **Avaddon Ransomware** (`ransomware.avaddon/delshad`).

<img width="1787" height="812" alt="Screenshot 2026-10-14 160225" src="https://github.com/user-attachments/assets/784c2f43-af39-40ff-8046-49105fe01eb6" />


### Step 3: Endpoint Security Inspection
* Inspected process execution logs for host `MarkPRD` (`172.16.17.88`) in **Endpoint Security**.
* **Finding:** Confirmed that `ab.exe` was actively executed on the system.
* **Suspicious Behaviors Detected:**
  * `ab.exe` spawned `wmic.exe` to delete Windows Volume Shadow Copies (`wmic shadowcopy delete`).
  * `ab.exe` spawned `bcdedit.exe` to disable Windows recovery options (`bcdedit /set {default} recoveryenabled No`).

<img width="1731" height="810" alt="Screenshot 2026-10-14 195259" src="https://github.com/user-attachments/assets/aab6f350-7f8e-43d3-9052-21beddd91d3f" />


### Step 4: Network Log & C2 Check
* Pivoted to **Log Management** to query network activity for IP `172.16.17.88` around May 23, 2021.
* **Finding:** No outbound HTTP/HTTPS proxy logs or Command and Control (C2) network connections were recorded. Impact was localized to local ransomware payload execution.

<img width="1750" height="382" alt="Screenshot 2026-10-14 195101" src="https://github.com/user-attachments/assets/7bf9a20e-41b5-4604-a9a6-bc58a741c573" />


---

## 📘 Playbook Execution & Incident Handling

### Action 1: Define Threat Indicator
* **Selection:** `Other`
* **Description:** The alert was triggered by local ransomware execution (`ab.exe`) on endpoint `MarkPRD` (`172.16.17.88`). "Other" was selected as it did not match network traffic, AV malfunction, or persistence startup categories.

<img width="625" height="215" alt="Screenshot 2026-10-14 214235" src="https://github.com/user-attachments/assets/3c828107-5174-4552-b60d-c867a5b344fc" />


### Action 2: Check Quarantine / Clean Status
* **Selection:** `Not Quarantined`
* **Description:** Checked Device Action metadata (`Allowed`) and process logs in Endpoint Security. Confirmed `ab.exe` ran successfully without automated interference, requiring manual intervention.

<img width="567" height="237" alt="Screenshot 2026-10-14 214709" src="https://github.com/user-attachments/assets/2ec497ad-3b31-4389-8feb-69594f10f606" />


### Action 3: Analyze Malware & C2 Check
* **Selection:** `Malicious` (Malware) | `Not Accessed` (C2)
* **Description:**
  * Analyzed binary hash on VirusTotal (57/66 detection consensus confirming Avaddon Ransomware).
  * Queried Log Management for external connections; no C2 communication was observed.
 
  <img width="532" height="302" alt="Screenshot 2026-10-14 215907" src="https://github.com/user-attachments/assets/a4313b23-7376-44bb-967d-7e14bd546298" />

<img width="590" height="292" alt="Screenshot 2026-10-14 222253" src="https://github.com/user-attachments/assets/7800df3b-4b2f-4e40-99b5-4e648ee9eebf" />


### Action 4: Host Containment
* **Action Taken:** Enabled **Host Containment** via EDR for `MarkPRD` (`172.16.17.88`).
* **Description:** Isolated the compromised host from the network to prevent lateral movement, file encryption spread across network shares, or potential C2 connection establishing later.

### Action 5: Add Artifacts
* **Description:** Recorded indicators of compromise (IoCs) to attach to the case record:
  * **MD5 Hash:** `0b486fe0503524cfe4726a4022fa6a68` (Avaddon Ransomware payload)
  * **Source IP:** `172.16.17.88` (Compromised Endpoint)
  * **Source Hostname:** `MarkPRD` (Target Windows Host)
  * **File Name:** `ab.exe` (Malicious Executable)
 
 <img width="625" height="377" alt="Screenshot 2026-10-14 223329" src="https://github.com/user-attachments/assets/292a571e-4bf0-45fd-8c41-e86f1afcd6e6" />


### Action 6: Write Analyst Note & Case Submission
* **Verdict:** `True Positive`
* **Description:** Documented full findings detailing local ransomware execution, shadow copy deletion, host containment actions, and closed the case as a confirmed True Positive.

<img width="652" height="323" alt="Screenshot 2026-10-14 223639" src="https://github.com/user-attachments/assets/027f6128-4e0d-46c4-ad8c-087c1a4bef23" />

---

## 💡 Conclusion & Final Remediation
* **Verdict:** True Positive
* **Reasoning:** Executable `ab.exe` was confirmed as Avaddon Ransomware based on 57/66 threat vendor detections and active host behavior attempting backup deletion (`wmic`, `bcdedit`).
* **Remediation Steps Taken:**
  1. Isolated host `MarkPRD` (`172.16.17.88`) from the network via EDR.
  2. Blocked hash `0b486fe0503524cfe4726a4022fa6a68` on endpoint protection platforms.
  3. Escalated to Tier 2 / Incident Response team for host forensic imaging, backup restoration, and credential reset.

  <img width="567" height="375" alt="Screenshot 2026-10-14 223725" src="https://github.com/user-attachments/assets/c53f3aa9-17df-4020-84a8-340f6745e842" />
