# 🛡️ LetsDefend Lab Write-Up: SOC105 - Requested T.I. URL Address

## 📌 Executive Summary
* **Platform:** LetsDefend
* **Alert Name:** SOC105 - Requested T.I. URL Address
* **Objective:** Investigate a security alert triggered by a URL linked to Threat Intelligence (T.I.) and determine whether it represents malicious activity or safe user behavior.

---

## 🔍 Initial Alert Details
* **EventID** :75
* **Event Time** :Mar, 07, 2021, 05:47 PM
* **Rule :SOC105** - Requested T.I. URL address
* **Level** :Security Analyst
* **Source Address** :10.15.15.12
* **Source Hostname** :MarksPhone
* **Destination Address** :67.199.248.10
* **Destination Hostname** :bit.ly
* **Username** :Mark
* **Request URL** :https://bit.ly/TAPSCAN
* **User Agent** :Mozilla/5.0 (Windows NT 5.1; Win64; x64) AppleWebKit/537.36 
(KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36
* **Device Action** :Allowed


<img width="656" height="386" alt="Screenshot 2026-10-13 231939" src="https://github.com/user-attachments/assets/2fe48da3-d1d7-49c9-8025-480bba39d626" />


---

## 📋 Step-by-Step Investigation Process

### Step 1: Analyzing the Alert Trigger
* The alert fired because the endpoint requested a URL (`bit.ly/TAPSCAN`) flagged by threat intelligence rules. 
* I began by reviewing the source host (`MarksPhone`) and user (`Mark`) to understand who initiated the request.

### Step 2: Destination IP & Domain Reputation Check
* I checked the destination IP address (`67.199.248.10`) on **AbuseIPDB**.
* **Finding:** The IP belongs to Bitly's legitimate content delivery network (CDN) infrastructure with a clean abuse score of 1%.

<img width="842" height="845" alt="Screenshot 2026-10-13 230221" src="https://github.com/user-attachments/assets/4ab7f17a-cdfe-44fd-bde4-b2fb0383eb16" />

### Step 3: VirusTotal Analysis
* I analyzed the shortened link (`https://bit.ly/TAPSCAN`) using **VirusTotal** to check for malicious indicators and community feedback.
* **Finding:** VirusTotal scans showed no malicious flags for the URL itself, confirming it pointed to a standard redirect service.

<img width="1826" height="922" alt="Screenshot 2026-10-13 084511" src="https://github.com/user-attachments/assets/83d31495-afc1-4553-8a5c-26a4074c99d1" />


### Step 4: urlscan.io Analysis
* I used **urlscan.io** to safely preview the behavior and redirection chain of the link without visiting it directly on a personal machine.
* **Finding:** The analysis showed that the shortened link safely expands through a standard analytics tracker directly to the official Google Play Store page for **"TapScanner"** (`pdf.tap.scanner`).

<img width="883" height="867" alt="Screenshot 2026-10-13 230820" src="https://github.com/user-attachments/assets/81d968a6-f1c7-483e-8d2b-ad33d61924c1" />
<img width="847" height="885" alt="Screenshot 2026-10-13 230852" src="https://github.com/user-attachments/assets/3ec90553-1696-409a-b992-d2fc0f669b27" />
 
### Step 5: Endpoint Inspection
* I reviewed endpoint logs for `MarksPhone` (10.15.15.12) to see if any secondary malicious actions occurred.
* **Finding:** No malicious payload downloads, unauthorized scripts, or suspicious file executions were found on the endpoint.
  
<img width="645" height="682" alt="Screenshot 2026-10-15 222254" src="https://github.com/user-attachments/assets/ffd20060-5664-488d-aa7d-69c84964eda7" />

---
## 📘 Playbook Execution & Incident Handling

### Action 1: Analyze Threat Intel Data
* **Selected Option:** `Non-malicious`
* **Description:** Checked the target URL and destination IP against threat intelligence tools (VirusTotal, AnyRun, urlscan.io, AbuseIPDB). Since all indicators showed clean results pointing to a legitimate Play Store app, the activity was classified as safe.

<img width="608" height="346" alt="Screenshot 2026-10-13 231954" src="https://github.com/user-attachments/assets/a0468f8e-36f5-4e05-b16e-486223fb112e" />

### Action 2: Add Artifacts
* **Description:** Recorded all relevant indicators collected during the investigation to attach to the incident record:
  * **Source IP:** `10.15.15.12` (Source Endpoint)
  * **Destination IP:** `67.199.248.10` (Bitly CDN)
  * **URL Address:** `https://bit.ly/TAPSCAN` (Shortened Redirect URL)
    
<img width="647" height="435" alt="Screenshot 2026-10-13 232404" src="https://github.com/user-attachments/assets/0a2a6237-a226-4918-82ed-11bdffd263fc" />


### Action 3: Write Analyst Note
* **Description:** Documented a clear summary of findings for future reference and audit trails:
  1. **IP Analysis:** Destination IP `67.199.248.10` verified as clean Bitly CDN infrastructure (1% abuse score).
  2. **URL Analysis:** URL expands safely to the official Google Play Store page for "TapScanner".
  3. **Endpoint Analysis:** No suspicious downloads or secondary compromises detected on `MarksPhone`.

<img width="678" height="385" alt="Screenshot 2026-10-13 232557" src="https://github.com/user-attachments/assets/4cc6c44c-afb8-400f-870b-4606b02b29ba" />

### Action 4: Case Closure & Verdict Submission
* **Selected Result:** `False Positive`
* **Description:** Confirmed and closed the case as a False Positive since no security threat or policy violation took place. No host containment was necessary.

<img width="672" height="292" alt="Screenshot 2026-10-13 232622" src="https://github.com/user-attachments/assets/70320cf2-89e1-4f90-a91f-1e466714c8da" />

---

## 💡 Conclusion & Final Action
* **Verdict:** False Positive
* **Reasoning:** The user clicked a legitimate shortened link (`bit.ly`) that redirected to a safe, official application page on the Google Play Store rather than a malicious command-and-control (C2) or malware hosting site.
* **Action Taken:** The alert was closed as non-malicious, and no host containment or system isolation was required.

