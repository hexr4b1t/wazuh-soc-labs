# Wazuh Setup – SOC Lab Foundation

This document covers the **end-to-end setup of a Wazuh SIEM lab**, which serves as the foundation for all subsequent detection and monitoring labs in this repository.

The goal of this setup is to deploy a **working Wazuh manager and dashboard**, onboard a **Windows agent**, and validate connectivity in a realistic SOC-style virtual environment.

---

## 1. Overview

### 1.1 What is Wazuh?
Wazuh is an open-source security monitoring platform used for:
- Log analysis
- File Integrity Monitoring (FIM)
- Intrusion detection
- Vulnerability detection
- Malware detection
- Compliance monitoring

It is widely used in **Security Operations Centers (SOC)** as a SIEM/XDR solution.


---

## 2. Lab Architecture

**Components:**
- Wazuh Manager + Dashboard (Ubuntu Server)
- Windows 10 Agent
- Host Windows Machine (for access and testing)

**Network Type:** Bridged Adapter

---

## 3. Prerequisites

### 3.1 VirtualBox
- Install Oracle VirtualBox on the host system
- Ensure the network adapter supports **Bridged Mode**

---

### 3.2 Ubuntu Server VM (Wazuh Manager)

**Recommended configuration:**
- OS: Ubuntu Server 22.04 LTS
- Disk: 50 GB
- RAM: 6–8 GB
- Network: Bridged Adapter
- IP Address: Static

#### Static IP Configuration
Configure a static IP using Netplan.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Example file:
```
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - x.x.x.x/24 <- static ip based on your subnet
      routes:
        - to: default
          via: x.x.x.x <-  your default gateway
      nameservers:
        addresses:
          - 1.1.1.1
          - 1.0.0.1
```

> [How to set static ip on ubuntu server](https://youtu.be/173rk-yiXKE?si=04uq5mP02xGCjIpw)

---

### 3.3 Windows 10 VM (Wazuh Agent)

**Recommended configuration:**
- Disk: 50 GB
- RAM: 6 GB
- Network: Bridged Adapter
- IP Address: Static

Ensure the Windows VM is on the **same subnet** as the Ubuntu server.
>[How to set static ip on windows](https://youtu.be/No9J23qW3JY?si=QqxIWk7oWiV64Fjf)
---

## 4. Installing Wazuh Manager (Ubuntu Server)

### 4.1 Add Wazuh GPG Key

Run the following command on the Ubuntu server:
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
```

This adds the GPG key required to verify Wazuh packages.

### 4.2 Install Wazuh Using the Installation Script
Download and execute the official installation script:
```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh sudo bash ./wazuh-install.sh -a -i
```


```-a``` → Installs all components (manager, indexer, dashboard)
```-i``` → Interactive mode

I am using version 4.12.

>⚠️ **Important**: Save the username and password displayed at the end of the installation.

---

## 5. Accessing the Wazuh Dashboard
Open a browser and navigate to: https://ubuntu-ip

Steps:
- Accept the browser security warning (self-signed certificate)
- Log in using the credentials provided during installation

---

## 6. Installing the Wazuh Agent (Windows VM)
### 6.1 Download the Agent
Download the Windows Wazuh Agent MSI from the [official documentation](https://documentation.wazuh.com/4.12/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)

>⚠️ **The agent version must match the Wazuh manager version.**

### 6.2 Install the Agent
- Run the MSI installer.
- Use default installation settings.
- Do not configure the agent yet.

---

## 7. Registering the Agent with the Manager
### 7.1 Generate Agent Key (Ubuntu Server)
```bash
Run sudo /var/ossec/bin/manage_agents
```

Steps:
1. Select A to add an agent
2. Agent name (e.g., windows-agent)
3. IP address (use static IP of windows)
4. Select E to extract the key
5. Copy the generated key

### 7.2 Apply Key on Windows Agent
- Open Wazuh Agent Manager from the Start Menu
- Paste the copied authentication key
- Enter the Wazuh manager IP address
- Save the configuration
- Restart the agent service

---

## 8. Verification
On the Ubuntu Server
```bash
sudo /var/ossec/bin/agent_control -lc
```

Expected output:

`ID: 000, Name: <manager>, Active/Local
`
`ID: 001, Name: windows-agent, IP: <windows-ip>, Active
`

On the Wazuh Dashboard
Agent appears as Active.
Status updates periodically.

---

## 9. Outcome
At the end of this lab setup:

1. Wazuh Manager and Dashboard are operational.✔️
2. Windows agent is successfully onboarded.✔️
3. The lab is ready for detection-focused experiments.✔️

This setup is reused across all subsequent SOC labs in this repository.


---
Disclaimer

This repository is for educational purposes only.
All tools and trademarks belong to their respective owners.
