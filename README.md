🛡️ Wazuh SIEM/FIM & Automated Threat Intelligence Lab

## 📖 Project Overview
This project demonstrates a complete, from-scratch deployment of **Wazuh SIEM**. It covers the provisioning of the central manager, endpoint agent onboarding, File Integrity Monitoring (FIM) configuration, and the integration of the **VirusTotal API** for automated, real-time malware detection and threat hunting.

---

## 🏗️ Phase 1: Wazuh Server Initialization

The first phase involves deploying the core Wazuh infrastructure, which includes the Indexer, Manager, Filebeat, and Dashboard.

### 1. Executing the Installation Script
The Wazuh all-in-one installation script was executed on the primary server. The terminal output confirms the successful sequential installation and configuration of the `wazuh-indexer`, `wazuh-manager`, `filebeat`, and `wazuh-dashboard`. 

Command to install (curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh && sudo bash ./wazuh-install.sh -a)

*Admin credentials were auto-generated at the end of the script.*
![Wazuh Installation Output]<img width="1717" height="961" alt="Screenshot 2026-07-19 172640" src="https://github.com/user-attachments/assets/80b638ac-b7a4-4b61-bf03-d1ddd9dfd553" />


### 2. Verifying Core Services
To ensure the infrastructure was stable, the status of the three primary daemon services was verified using `systemctl`. All services reported as `active (running)`.

**Wazuh Manager Status:**
![Wazuh Manager Status]<img width="1717" height="962" alt="Screenshot 2026-07-19 173910" src="https://github.com/user-attachments/assets/4b36f02f-6c0e-4857-b37f-6b8471bc005a" />


**Wazuh Indexer Status:**
![Wazuh Indexer Status]<img width="1636" height="543" alt="Screenshot 2026-07-19 174025" src="https://github.com/user-attachments/assets/6b6d2dba-0598-48c7-aef2-f2fafc6559fa" />


**Wazuh Dashboard Status:**
![Wazuh Dashboard Status]<img width="1641" height="517" alt="Screenshot 2026-07-19 174114" src="https://github.com/user-attachments/assets/b56e0c5e-5565-4a34-8c99-54eefe97f611" />


---

## 📡 Phase 2: Agent Deployment (Kali Linux Endpoint)

With the server running, a Linux endpoint was onboarded to forward logs and telemetry to the SIEM.

### 1. Package Download and Installation
Setting the The Wazuh agent on Kali linux Debian package (`v4.14.6`) was downloaded via `wget` and unpacked using `dpkg`. The `WAZUH_MANAGER` environment variable was passed inline to point the agent to the central server.
to install Agent (wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.6-1_amd64.deb)  &&

to link the agent to your Server  (sudo WAZUH_MANAGER='<WAZUH_MANAGER_IP>' dpkg -i ./wazuh-agent_4.14.6-1_amd64.deb)

![Agent Download and DPKG]<img width="1170" height="649" alt="Screenshot 2026-07-19 183506" src="https://github.com/user-attachments/assets/1e9e03f2-54fb-4460-809b-d11553d07e86" />


### 2. Service Activation
The system daemon was reloaded, and the agent was enabled and started. Checking `systemctl status wazuh-agent` confirmed the agent modules (execd, agentd, syscheckd, logcollector) were spawned successfully.
![Agent Systemctl Status]<img width="1020" height="654" alt="Screenshot 2026-07-19 185835" src="https://github.com/user-attachments/assets/42bd6065-e6e3-419c-a104-62212c001811" />


### 3. Dashboard Verification
Inside the Wazuh Web UI, the **Endpoints Summary** confirmed the agent (`ID: 001`) successfully registered and checked in with an `active` status.
![Agent Active on Dashboard]
<img width="1275" height="727" alt="Screenshot 2026-07-26 195315" src="https://github.com/user-attachments/assets/862886b7-2330-492c-b342-d7adb4c53499" />
Full Interface of Active Agent
<img width="1908" height="959" alt="Screenshot 2026-07-19 185802" src="https://github.com/user-attachments/assets/4c5d539f-e107-489a-b3f8-fce89c825f52" />





---

## 🔍 Phase 3: File Integrity Monitoring (FIM)

FIM (Syscheck) was configured to monitor a specific user directory for any unauthorized file modifications.

### 1. Modifying the Agent Configuration
The `/var/ossec/etc/ossec.conf` file on the Kali endpoint was edited to add the target directory under the `<syscheck>` block, enforcing `realtime="yes"` and `check_all="yes"`.
![Editing ossec.conf for FIM]<img width="919" height="631" alt="Screenshot 2026-07-19 192636" src="https://github.com/user-attachments/assets/8cce13bc-eb11-4fa6-9b33-7815bffbf097" />


### 2. Baseline Testing
A test file (`fim_demo.txt`) was created, modified, and removed via terminal to generate baseline FIM telemetry.
![FIM Terminal Test]<img width="1215" height="689" alt="Screenshot 2026-07-26 212557" src="https://github.com/user-attachments/assets/ef513a1b-ba00-4c6a-9016-1d0e6c42c3f9" />


### 3. FIM Dashboard Analytics
The Wazuh FIM Dashboard successfully captured the events, populating the visual metrics for **Files added**, **Files modified**, and **Files deleted**.
![FIM Dashboard Overview]<img width="1283" height="907" alt="Screenshot 2026-07-26 195726" src="https://github.com/user-attachments/assets/c88eef39-7f3a-4459-908d-a9a6301c1299" />


A detailed review of the **Events** tab showed the exact timestamps and rule levels triggered by the `fim_demo.txt` and `.zsh_history` modifications. A compliance report was also generated.
![FIM Events Tab]
<img width="1279" height="921" alt="Screenshot 2026-07-26 195832" src="https://github.com/user-attachments/assets/fc148ea8-d005-4cb9-8c97-e4a4f632c0e8" />

![FIM PDF Report]<img width="792" height="799" alt="Screenshot 2026-07-26 195947" src="https://github.com/user-attachments/assets/80cc6503-1c9c-4bac-9ca0-3fb9e8be3f65" />


---

## 🦠 Phase 4: VirusTotal Threat Intelligence Integration

To automate malware analysis, the Wazuh Manager was linked to the VirusTotal API to scan file hashes captured by the FIM module.

### 1. API Provisioning
A standard public API key was generated via the VirusTotal portal.
![VirusTotal Home]<img width="1282" height="915" alt="Screenshot 2026-07-26 201553" src="https://github.com/user-attachments/assets/d59d0356-3b7e-47ef-a1cf-ff2c7879f298" />

![VirusTotal API Key]<img width="1278" height="910" alt="Screenshot 2026-07-26 201612" src="https://github.com/user-attachments/assets/b2e8bcf1-2d37-48b5-bdee-4d5c00839121" />


### 2. Manager Configuration
The `/var/ossec/etc/ossec.conf` file on the **Wazuh Server** was edited. An `<integration>` block was added, passing the VirusTotal API key and binding it to the `syscheck` group.
![Manager VirusTotal Integration]<img width="854" height="441" alt="Screenshot 2026-07-26 212648" src="https://github.com/user-attachments/assets/92857306-4858-405d-bfe3-c27bce115a50" />


### 3. Restarting the Manager
The `wazuh-manager` service was restarted to load the new integration. The `systemctl` status confirmed the `wazuh-integratord` process was running.
![Manager Restart and Integratord]<img width="857" height="472" alt="Screenshot 2026-07-26 212807" src="https://github.com/user-attachments/assets/43b14525-9fa6-41f1-a6fb-6632bec6d249" />


---

## 🚨 Phase 5: Threat Simulation & Hunting

### 1. Malicious Payload Simulation
On the endpoint, the standard **EICAR anti-virus test string** was piped into a new file (`eicar.com`) inside the FIM-monitored directory. IN KALI-LINUX
![EICAR Creation]<img width="1219" height="759" alt="Screenshot 2026-07-26 212540" src="https://github.com/user-attachments/assets/92b5d97e-16e6-499a-a4aa-5e2ca4938b69" />


### 2. Automated Threat Detection
Wazuh's FIM module instantly detected the file creation, extracted the hash, and passed it to the VirusTotal integration. VirusTotal identified the hash as malicious across 62 security engines.

This triggered a critical **Level 12 Alert** in the Wazuh Threat Hunting Dashboard.
![Threat Hunting Level 12 Alert]<img width="1267" height="929" alt="Screenshot 2026-07-26 213109" src="https://github.com/user-attachments/assets/340de990-12c7-4def-a5c0-99edd8bccce9" />


Expanding the alert details revealed the exact rule description: `VirusTotal: Alert - /home/<USER>/eicar.com - 62 engines detected this file`.
![VirusTotal Alert Details]<img width="1254" height="814" alt="Screenshot 2026-07-26 214356" src="https://github.com/user-attachments/assets/cf4a469d-11d1-47ce-b9a7-38c6bf626c5b" />

<img width="1253" height="812" alt="Screenshot 2026-07-26 214413" src="https://github.com/user-attachments/assets/4685572b-9d33-4888-8913-494c000858f5" />



A final Threat Hunting PDF report was generated, summarizing the high-severity malware detection alongside standard authentication failures.
![Threat Hunting PDF Report]<img width="884" height="807" alt="Screenshot 2026-07-26 212955" src="https://github.com/user-attachments/assets/1a4a96bf-3709-424c-88e5-e4135af4b7e7" />

