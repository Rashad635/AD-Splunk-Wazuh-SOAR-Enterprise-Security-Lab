# AD + Splunk + Wazuh + SOAR Enterprise Security Lab
**Date: July 31, 2026**

An end-to-end cybersecurity home lab demonstrating enterprise-style security monitoring, detection engineering, threat intelligence enrichment, and SOAR automation using Active Directory, Splunk, Wazuh, Sysmon, Tines, VirusTotal, AbuseIPDB, and Slack.

The project simulates a small enterprise SOC environment where controlled security activity is generated from Kali Linux, collected from Windows and Linux systems, analyzed through Wazuh and Splunk, enriched through threat intelligence platforms, and automated through Tines SOAR.

---

## Architecture

<img width="850" height="1100" alt="AD_Splunk_Wazuh_SOAR" src="https://github.com/user-attachments/assets/60731d25-52fe-4f81-9e5b-f13cb8b83268" />

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

The network contains:
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

5. Ubuntu security server
The Ubuntu server provides the central security infrastructure.
Ubuntu Server
192.168.10.40
NAT: 192.168.169.135
Services hosted on the server:
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
Wazuh responsibilities
Wazuh is responsible for:
Endpoint monitoring
Windows event collection
Sysmon telemetry
Linux monitoring
File Integrity Monitoring
Security detection
Custom detection rules
MITRE ATT&CK mapping
Alert generation
Agent management
Splunk responsibilities
Splunk provides:
Centralized event indexing
SPL-based investigation
Windows event analysis
Sysmon analysis
Security dashboards
Correlation searches
Detection development
Historical event investigation
Running both platforms provides the ability to investigate the same endpoint activity from two SIEM perspectives.

6. Active Directory domain controller
The Domain Controller is the identity and authentication core of the lab.
Domain Controller
192.168.10.50

+-- Active Directory
+-- DNS
+-- Windows Security Events
Domain:
local.net
Example domain systems:
fd-pc1.local.net
dp-pc1.local.net
The Domain Controller provides:
User accounts
Computer accounts
Authentication
Kerberos
Group Policy
DNS
Domain membership
Windows security event generation
Important security telemetry includes:
Successful logons
Failed logons
Account creation
Account deletion
Account modifications
Privilege changes
Authentication failures
Suspicious account activity

7. Windows endpoints
The lab contains two Windows endpoints.
FD-PC1
Host IP: 192.168.10.30
NAT IP:  192.168.169.184
DP-PC1
Host IP: 192.168.10.20
NAT IP:  192.168.169.226
Each endpoint contains:
Windows Endpoint
|
+-- Sysmon
|
+-- Wazuh Agent
|
+-- Splunk Universal Forwarder
This creates two telemetry pipelines.
Wazuh telemetry
Windows
   |
Sysmon + Windows Events
   |
Wazuh Agent
   |
Wazuh Manager
   |
Wazuh Indexer
   |
Wazuh Dashboard
Splunk telemetry
Windows
   |
Sysmon + Windows Events
   |
Splunk Universal Forwarder
   |
Splunk Indexer
   |
Splunk Enterprise

8. Sysmon
Sysmon provides detailed Windows endpoint telemetry that can be used for threat detection and incident investigation.
Important Sysmon events include:
Event ID
Activity
1
Process creation
3
Network connection
7
Image/DLL loading
10
Process access
11
File creation
17
Named pipe creation
22
DNS query

The lab uses Sysmon to collect telemetry such as:
Process creation
Network connections
File creation
Registry activity
Process termination
DNS queries



