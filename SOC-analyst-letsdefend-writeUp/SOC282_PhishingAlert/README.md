# 🛡️ LetsDefend Lab Write-Up: SOC282 - Phishing Alert (Deceptive Mail Detected)

## 📌 Executive Summary
* **Platform:** LetsDefend[cite: 18]
* **Alert Name:** SOC282 - Phishing Alert - Deceptive Mail Detected[cite: 18]
* **Event ID:** 257[cite: 18]
* **Rule Name:** SOC282 - Phishing Alert - Deceptive Mail Detected[cite: 18]
* **Alert Type:** Exchange[cite: 18]
* **Difficulty / Severity:** Medium[cite: 18]
* **MITRE ATT&CK Techniques:**
  * `T1566` - Initial Access: Phishing[cite: 18]
  * `T1566.002` - Initial Access: Spearphishing Link[cite: 18]
  * `T1059` - Execution: Command and Scripting Interpreter[cite: 18]
  * `T1204` - Execution: User Execution[cite: 18]

---

## 🔍 Initial Alert Details
* **Event Time:** May 13, 2024, 09:22 AM (`2024-05-13T09:22:00+03:00`)
* **SMTP Address:** 103.80.134.63
* **Source Address:** `free@coffeeshooop.com`
* **Destination Address:** `Felix@letsdefend.io`
* **E-mail Subject:** Free Coffee Voucher
* **Device Action:** Allowed

<img width="655" height="606" alt="Screenshot 2026-10-14 225156" src="https://github.com/user-attachments/assets/e973c9e9-65f1-41e5-9493-e5fe68a8c78c" />

---

## 📋 Step-by-Step Investigation Process

### Step 1: Email Security Review
* **Action:** Searched for the sender address (`free@coffeeshooop.com`) in **Email Security**.
* **Social Engineering Tactics:** Uses a promotional incentive ("Free Cup of Coffee") and artificial urgency ("Hurry, this offer expires soon!") to trick the recipient into clicking the embedded redemption link or opening the archive attachment (`free-coffee.zip`).

<img width="1130" height="627" alt="Screenshot 2026-10-14 235004" src="https://github.com/user-attachments/assets/885f9663-e127-4718-839d-34e3a93ae672" />


### Step 2: Log Management Analysis
* **Action:** Searched proxy logs in **Log Management** for host activity related to the download.
* **Key Finding:** Proxy logs confirm user interaction. The victim (`Felix`) clicked the phishing link inside the email, causing Google Chrome (`chrome.exe`) to successfully connect to an AWS S3 bucket and download the malicious zip archive (`free-coffee.zip`).

<img width="937" height="492" alt="Screenshot 2026-10-15 005025" src="https://github.com/user-attachments/assets/40821c32-4efd-4c89-bb60-63654ff9607e" />


### Step 3: Threat Intelligence Analysis (SMTP Server)
* **Action:** Analyzed the sender SMTP IP address (`103.80.134.63`) using **VirusTotal**.
* **Target IP:** `103.80.134.63` (LG DACOM Corporation, South Korea)
* **VirusTotal Reputation:** 7/92 security vendors flagged the IP address as **Malicious / Phishing**.
* **Analysis:** The sender IP address used to deliver the email was confirmed to have a negative threat reputation associated with phishing campaigns, validating the initial alert trigger for deceptive mail.

<img width="1890" height="666" alt="Screenshot 2026-10-14 235733" src="https://github.com/user-attachments/assets/4a85529c-cb5b-450e-84c7-5cdc1410083b" />

### Step 4: Attachment Sandbox & Threat Intelligence Analysis (Hybrid Analysis)
* **Artifact:** `free-coffee.zip`
* **Platform:** Hybrid Analysis
* **Threat Score:** 100/100 (MALICIOUS)
* **Threat Classification:** Identified as `Trojan.Generic` / `AsyncRAT` / `BLIND SPIDER` malware campaign.
* **Detection Summary:** Antivirus engines (including Bitdefender and Emsisoft) classified the compressed payload as a malicious Trojan (`Trojan.GenericKD.73385324`).
* **Key Hashes:**
  * **MD5:** `73f0f77181e1..9e7a03fe`
  * **SHA-256:** `6f33ae4bf134..ab2fb389`
 
<img width="805" height="430" alt="Screenshot 2026-10-15 001737" src="https://github.com/user-attachments/assets/cda2d875-48bd-43ea-95aa-2fd11eaa6aa2" />
<img width="782" height="457" alt="Screenshot 2026-10-15 001843" src="https://github.com/user-attachments/assets/d28740a3-55ae-4286-9e8c-16683a8c91b1" />


### Step 5: Sandbox Dynamic Analysis (ANY.RUN)
* **Archive Extraction:** The archive `free-coffee.zip` was opened via `WinRAR.exe` (PID 4788), extracting `Coffee.exe`.
* **Payload Execution:** `Coffee.exe` (PID 8012) executed from `C:\Users\admin\AppData\Local\Temp\...`.
* **Malware Family Identification:** Flagged with a **100/100 Threat Score** as **AsyncRAT** via explicit Mutex detection (`ASYNCRAT has been detected (MUTEX)`).
* **Defense Evasion:** `Coffee.exe` uses binary masquerading, falsely claiming Microsoft metadata (`COM Surrogate`, `10.0.19041.546`).
* **Observed MITRE ATT&CK TTPs:**
  * `T1204.002` (User Execution): Payload executed manually from extracted archive.
  * `T1082` (System Information Discovery): Queries registry keys for machine GUID, computer name, and system language settings.
  * `T1012` (Query Registry): System registry enumeration for environment profiling.

<img width="797" height="533" alt="Screenshot 2026-10-15 002515" src="https://github.com/user-attachments/assets/6755f609-143e-42a8-af93-fbab76d41f2f" />
<img width="787" height="492" alt="Screenshot 2026-10-15 002547" src="https://github.com/user-attachments/assets/1822b6c6-483d-462f-9699-8acd063ccc05" />
<img width="782" height="490" alt="Screenshot 2026-10-15 002650" src="https://github.com/user-attachments/assets/84500bb3-114e-4720-9e3f-2c0f7b4c2729" />


### Step 6: VirusTotal Threat Intelligence (Zip Archive)
* **Artifact:** `free-coffee.zip`
* **SHA-256 Hash:** `6f33ae4bf134c49faa14517a275c039ca1818b24fc2304649969e399ab2fb389`
* **Detection Score:** 18/63 security vendors flagged the sample as malicious.
* **Threat Classification:** Identified under the threat labels `trojan.znyonm` and `asyncrat`. Static behavior tags highlight anti-analysis techniques including debugger detection (`detect-debug-environment`), execution delay (`long-sleeps`), and payload obfuscation (`obfuscated`).

<img width="725" height="456" alt="Screenshot 2026-10-15 002828" src="https://github.com/user-attachments/assets/94f231b9-4672-4963-b9f8-6c9c97d66e10" />

