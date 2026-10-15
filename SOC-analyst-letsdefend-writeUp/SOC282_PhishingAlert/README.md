# 🛡️ LetsDefend Lab Write-Up: SOC282 - Phishing Alert (Deceptive Mail Detected)

## 📌 Executive Summary
* **Platform:** LetsDefend
* **Alert Name:** SOC282 - Phishing Alert - Deceptive Mail Detected
* **Event ID:** 257
* **Rule Name:** SOC282 - Phishing Alert - Deceptive Mail Detected
* **Alert Type:** Exchange
* **Difficulty / Severity:** Medium
* **MITRE ATT&CK Techniques:**
  * `T1566` - Initial Access: Phishing
  * `T1566.002` - Initial Access: Spearphishing Link
  * `T1059` - Execution: Command and Scripting Interpreter
  * `T1204` - Execution: User Execution

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

---

## 📘 Playbook Execution & Incident Handling

### Action 1: Parse Email Information
* **When was it sent?:** May 12, 2024, 23:22:00 (or May 13, 2024, 09:22 AM per alert header)
* **Email's SMTP Address:** `103.80.134.63`
* **Sender Address:** `free@coffeeshooop.com`
* **Recipient Address:** `Felix@letsdefend.io`
* **Mail Content Suspicious?:** Yes (Typosquatted domain, urgency tactics, external download links)
* **Attachments?:** Yes (`free-coffee.zip`)
<img width="583" height="305" alt="Screenshot 2026-10-15 005519" src="https://github.com/user-attachments/assets/6fc221eb-ca3d-49ca-8991-c2c4bfd21d63" />


### Action 2: Check for Attachments or URLs
* **Selected Option:** `Yes`
* **Attachment:** `free-coffee.zip`
* **Embedded URL:** The "Redeem Now" link leading to the external AWS S3 bucket.
<img width="586" height="258" alt="Screenshot 2026-10-15 005611" src="https://github.com/user-attachments/assets/ab95e118-3c9b-4b9b-9c11-20a28d302c0e" />


### Action 3: Analyze Url / Attachment
* **Selected Option:** `Malicious`
* **Hybrid Analysis Score:** 100/100 (MALICIOUS), flagged as `AsyncRAT` / `BLIND SPIDER`.
* **ANY.RUN Sandbox Result:** 100/100 Threat Score, detected `Coffee.exe` dropped from `free-coffee.zip` with explicit AsyncRAT MUTEX detection.
* **VirusTotal Detections:** 18/63 security vendors flagged `free-coffee.zip` as malicious.
<img width="556" height="335" alt="Screenshot 2026-10-15 005703" src="https://github.com/user-attachments/assets/628b5d08-07af-41ad-92ee-693010bccb37" />


### Action 4: Check If Mail Delivered to User
* **Selected Option:** `Delivered`
* **Reasoning:** Based on the initial alert details and email security record (`Action: Allowed`), the phishing email was successfully delivered directly to the recipient's inbox (`Felix@letsdefend.io`) without being blocked by mail filters.

<img width="565" height="183" alt="Screenshot 2026-10-15 005804" src="https://github.com/user-attachments/assets/344e0c0e-82c0-41d7-93a7-a78db357e0ed" />


### Action 5: Delete Email From Recipient
* **Action Taken:** Clicked `Delete`.
* **Reasoning:** Since the phishing email landed successfully in the user's inbox (`Felix@letsdefend.io`) and contains malicious links and payloads, standard incident response remediation requires purging/deleting the malicious email from the user's mailbox to prevent re-opening.
<img width="580" height="175" alt="Screenshot 2026-10-15 010022" src="https://github.com/user-attachments/assets/9d369755-2c7d-4d68-af17-e0d546bfa2f8" />


### Action 6: Check If Someone Opened the Malicious File/URL
* **Selected Option:** `Opened`
* **Reasoning:** Proxy and endpoint logs confirmed that the user (`Felix`) successfully accessed the external S3 bucket URL via Google Chrome and downloaded the malicious archive (`free-coffee.zip`). Subsequent endpoint monitoring verified that the compressed payload was extracted and executed (`Coffee.exe`), establishing outbound communication with the threat actor's Command and Control (C2) infrastructure. Therefore, the artifact was confirmed as Opened.
<img width="527" height="276" alt="Screenshot 2026-10-15 010107" src="https://github.com/user-attachments/assets/2881692d-5795-4525-805c-31fa0846f9aa" />

<img width="937" height="492" alt="Screenshot 2026-10-15 005025" src="https://github.com/user-attachments/assets/e5530b12-bb5d-4e60-881b-c4d6a90441cb" />
<img width="785" height="397" alt="Screenshot 2026-10-15 011726" src="https://github.com/user-attachments/assets/89cabc4d-424f-4810-b913-ffda7f32645d" />


### Action 7: Host Containment
* **Action Taken:** Isolated the compromised workstation (`Felix` / `172.16.20.151`) via the EDR containment toggle to prevent lateral movement and C2 communication.

<img width="582" height="177" alt="Screenshot 2026-10-15 012133" src="https://github.com/user-attachments/assets/6975492d-30fc-490e-98fd-ca674c7f5c81" />
<img width="640" height="358" alt="Screenshot 2026-10-15 012259" src="https://github.com/user-attachments/assets/cc2d9782-2ff4-4347-b665-544c10ae9fcf" />


### Action 8: Add Artifacts
* **Recorded Indicators of Compromise (IoCs):**
  * **103.80.134.63:** Originating IP address
  * **AWS S3 Bucket URL:** `https://files-ld.s3.us-east-2.amazonaws.com/59cbd215-76ea-434d-93ca-4d6aec3bac98-free-coffee.zip`
  * **6f33ae4bf134c49faa14517a275c039ca1818b24fc2304649969e399ab2fb389:** Cryptographic hash of file
  * **172.16.20.151:** Internal IP address of host
<img width="656" height="382" alt="Screenshot 2026-10-15 013000" src="https://github.com/user-attachments/assets/481e5541-1383-4616-b623-424140649e7d" />


### Action 9: Analyst Note & Case Submission
* **Analyst Note Summary:** Investigated phishing alert (Event ID 257) targeting host `Felix` (`172.16.20.151`). Confirmed email delivery from typosquatted domain `coffeeshooop.com` containing a malicious AWS S3 download link (`free-coffee.zip`). Proxy and EDR logs verified that the user downloaded, extracted, and executed the payload (`coffee.exe`), which was confirmed via sandbox analysis to be AsyncRAT initiating outbound C2 communication to `37.120.233.226`. Remediation actions completed successfully: malicious email purged from mailbox, host isolated via EDR, and IoCs documented for blocking. Case closed as True Positive.
* **Final Verdict:** `True Positive`

<img width="607" height="316" alt="Screenshot 2026-10-15 013053" src="https://github.com/user-attachments/assets/154bb37f-2e5c-4763-82d9-f0014ba2e14d" />

<img width="543" height="210" alt="Screenshot 2026-10-15 013110" src="https://github.com/user-attachments/assets/9b6e0e90-17fa-4915-b2da-df3d4512a21f" />


---

## 💡 Conclusion & Remediation Summary
* **Verdict:** True Positive
* **Reasoning:** Confirmed delivery and user execution of an AsyncRAT malware payload delivered via a spearphishing link and typosquatted email domain.
* **Remediation Completed:**
  1. Purged phishing email from user's mailbox (`Felix@letsdefend.io`).
  2. Isolated compromised host (`172.16.20.151`) via EDR.
  3. Documented IoCs (`103.80.134.63`, file hash, S3 download URL, C2 IP `37.120.233.226`) for network-wide perimeter blocking.
