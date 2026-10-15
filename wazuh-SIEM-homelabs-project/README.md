# 🛡️ Real-Time Endpoint File Integrity Monitoring & Threat Intelligence via Wazuh

![Wazuh](https://img.shields.io/badge/Wazuh-SIEM%20Syscheck-blue?style=for-the-badge&logo=wazuh)
![Windows 11](https://img.shields.io/badge/Target-Windows%2011-0078D4?style=for-the-badge&logo=windows)
![Ubuntu](https://img.shields.io/badge/Manager-Ubuntu%20Linux-E95420?style=for-the-badge&logo=ubuntu)
![VirusTotal](https://img.shields.io/badge/CTI-VirusTotal%20API-396afc?style=for-the-badge&logo=virustotal)

---

## 1. 📌 Project Title & Overview
* **Project Title:** Real-Time Endpoint File Integrity Monitoring (FIM) & Automated Threat Intelligence via VirusTotal
* **Objective:** Deploy and configure the Wazuh SIEM `syscheck` module to perform real-time File Integrity Monitoring (FIM) on a Windows 11 endpoint, integrating VirusTotal API threat intelligence and automated Active Response to detect and purge malicious files (`EICAR`) instantaneously.

---

## 2. 🏗️ Infrastructure Architecture & Initial Environment Setup

Before implementing Threat Intelligence and Active Response, a full SOC lab environment was built consisting of a central SIEM Manager and an Endpoint Agent.

```text
┌──────────────────────────────────────┐       Log Telemetry & FIM Events      ┌────────────────────────────────────────┐
│         Ubuntu Server (VM)           │ ────────────────────────────────────► │            Windows 11 (VM)             │
│  Wazuh Manager & Wazuh Dashboard     │                                       │          Wazuh Endpoint Agent          │
│          IP: 192.168.137.129         │ ◄──────────────────────────────────── │          IP: 192.168.137.131           │
└──────────────────────────────────────┘       Active Response Execution       └────────────────────────────────────────┘
