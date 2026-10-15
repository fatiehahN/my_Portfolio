# Wazuh SIEM Lab Setup & Security Monitoring

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
```

### Phase 1: Wazuh Manager & Dashboard Deployment (Ubuntu Linux)

1. **Virtual Machine Provisioning:**
   * Installed Ubuntu Server on VMware Workstation.
   * Configured network adapter to Host-Only / NAT mode to ensure stable local connectivity (`192.168.137.129`).

2. **Wazuh Central Stack Installation:**
   * Downloaded and executed the official Wazuh installation script on the Ubuntu terminal to deploy the Wazuh Manager, Indexer, and Dashboard:
     ```bash
     curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh)
     sudo bash wazuh-install.sh -a
     ```
   * Saved the generated administrator password for Dashboard access.

3. **Service Verification:**
   * Verified that the Wazuh Manager service was active and listening on central log collection ports (`1514` for agent traffic and `55000` for API):
     ```bash
     sudo systemctl status wazuh-manager
     ```
   * Accessed the Wazuh Dashboard web GUI by navigating to `https://192.168.137.129` in the browser.
  

### Phase 2: Wazuh Agent Deployment (Windows 11 Endpoint)

1. **Agent Installation:**
   * Downloaded the official Windows Wazuh Agent installer (`.msi`) onto the Windows 11 target machine (`Windows-VM-Agent`).
   * Ran the installer via Administrator PowerShell, specifying the Ubuntu Manager IP address:
     ```powershell
     msiexec.exe /i wazuh-agent-4.x.x.msi /q WAZUH_MANAGER='192.168.137.129' WAZUH_REGISTRATION_SERVER='192.168.137.129'
     ```

2. **Agent Service Activation:**
   * Started the Wazuh service on Windows to establish connection with the manager:
     ```powershell
     NET START Wazuh
     ```

3. **Connection Verification:**
   * Checked `C:\Program Files (x86)\ossec-agent\ossec.log` on Windows to confirm the successful handshake message:
     ```text
     INFO: Connected to manager at 192.168.137.129:1514
     ```
   * Verified on the Wazuh Dashboard under **Management** $\rightarrow$ **Agents** that `Windows-VM-Agent` appeared with an **Active** status.
