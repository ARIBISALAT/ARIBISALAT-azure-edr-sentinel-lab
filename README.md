# 🛡️ End-to-End EDR & SIEM Detection Engineering Lab
### LimaCharlie EDR Integration with Microsoft Sentinel & Atomic Red Team Simulation

![Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-blue?logo=microsoftazure)
![Sentinel](https://img.shields.io/badge/SIEM-Microsoft%20Sentinel-0078D4?logo=microsoft)
![LimaCharlie](https://img.shields.io/badge/EDR-LimaCharlie-FF6600)
![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE%20ATT%2CK-red)

---

## 📌 Executive Summary

This project demonstrates the end-to-end implementation, configuration, and validation of a cloud-native SOC detection engineering environment. By deploying **LimaCharlie EDR** agents across hybrid virtual endpoints hosted in **Microsoft Azure**, real-time endpoint telemetry was ingested into a **Microsoft Sentinel** Log Analytics Workspace. 

To evaluate the detection pipeline, controlled adversary simulations based on **Atomic Red Team** principles were executed against Windows and Linux target virtual machines. The results confirm real-time process monitoring, Syslog telemetry ingestion, and live incident generation inside Microsoft Sentinel.

---

## 🏗️ Lab Architecture & Infrastructure

* **Cloud Provider:** Microsoft Azure
* **Target Endpoints:**
  * `vm-win-01` — Windows Server / Endpoint Target
  * `vm-linux-01` — Linux Target Endpoint
* **EDR Agent:** LimaCharlie EDR Sensor (`rphcpsvc` / cross-platform agent)
* **SIEM Platform:** Microsoft Sentinel (Workspace: `veenodelabs`)
* **Telemetry Forwarding:** Syslog / CEF JSON pipeline to Log Analytics

---

## 🚀 Adversary Simulation Execution (Atomic Red Tests)

Controlled adversary techniques mapped to the **MITRE ATT&CK** framework were executed via Azure **Run Command** administrative interfaces to generate host-level behavioral telemetry.

### 1. 🪟 Windows Target (`vm-win-01`)
* **Objective:** Simulate System Discovery and Persistence via Service Creation.
* **Commands Executed:**
  ```powershell
  # System & User Enumeration (T1082 / T1033)
  systeminfo
  whoami /all
  net user

  # Service Creation Persistence Simulation (T1543.003)
  sc.exe create TestService binPath= "C:\Windows\System32\svchost.exe" start= auto
  sc.exe delete TestService
2. 🐧 Linux Target (vm-linux-01)
Objective: Simulate Sensitive Credential Access (OS Credential Dumping).

Commands Executed:

Bash
# OS Credential Access Simulation (T1003.008)
sudo cat /etc/shadow
🔄 Telemetry Pipeline & Detection Flow
[ Target VM Endpoint ] 
        │
        ├──> (1) Execution: Atomic test command run via Azure Run Command
        │
[ LimaCharlie EDR Sensor ] 
        │
        ├──> (2) EDR Interception: Real-time event logged in Timeline (NEW_PROCESS / TERMINATE_PROCESS)
        │
[ Ingestion Pipeline ]
        │
        ├──> (3) Log Ingestion: Syslog / CEF telemetry forwarded to Azure Log Analytics
        │
[ Microsoft Sentinel ]
        └──> (4) Incident Triggered: ID 423 / ID 424 generated ("New Service Installed")
📸 Evidence & Proof of Work
A. Real-Time EDR Telemetry (LimaCharlie)
The LimaCharlie sensor running on vm-linux-01 captured real-time endpoint activity, recording execution lifecycle events (TERMINATE_PROCESS, process creation, and PIDs) inside the sensor timeline.

B. SIEM Incident Generation (Microsoft Sentinel)
Telemetry successfully crossed the ingestion pipeline, triggering live active incidents inside the Microsoft Sentinel portal.
<img width="950" height="386" alt="limacharlie-linux-timeline png" src="https://github.com/user-attachments/assets/3e75aa3d-b372-4de4-82df-646438fda374" />


1. Incident ID 423: New Service Installed
Shows initial EDR agent service registration and execution tracking assigned within the veenodelabs workspace.
<img width="1271" height="503" alt="sentinel-incident-423 png" src="https://github.com/user-attachments/assets/bfc353eb-f77f-400d-8520-a265d1f58c3c" />


3. Incident ID 424: New Service Installed
Demonstrates live incident generation resulting from active persistence simulation (sc.exe create) on the target host.
<img width="1189" height="532" alt="sentinel-incident-424 png" src="https://github.com/user-attachments/assets/7c734209-b929-4aed-9bcc-8243a8b2441b" />


C. Log Analytics & Ingestion Health Verification
To confirm continuous telemetry stream health, KQL queries were executed inside Log Analytics to verify real-time log ingestion across enrolled endpoints.
<img width="933" height="443" alt="kql-log-verification" src="https://github.com/user-attachments/assets/c1b0827e-4e69-4126-978a-f023f7cf7755" />


Code snippet
// Verify real-time Syslog ingestion from EDR agents
Syslog
| where TimeGenerated > ago(24h)
| project TimeGenerated, Computer, SyslogMessage
🛠️ Key Takeaways & SOC Operations Impact
End-to-End Visibility: Validated full visibility from low-level host process creation (LimaCharlie EDR) up to top-level cloud SIEM incident generation (Microsoft Sentinel).


Detection Engineering: Confirmed that host-based service creation (sc.exe) reliably populates actionable alerts mapped to MITRE ATT&CK T1543.003.

Log Ingestion Health: Verified active Syslog telemetry streaming across Linux and Windows assets, providing a scalable foundation for custom KQL detection rules.
