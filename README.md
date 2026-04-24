# Active Directory Lab with Splunk SIEM & AI Automation

A home SOC environment built on AWS simulating a real enterprise Active Directory domain. Windows event logs from a Domain Controller and client machine are ingested into a Splunk SIEM, where custom detection rules trigger automated AI-powered alerts delivered to Discord in real time.

---

## Overview

This project simulates a mini enterprise environment with a full detection and alerting pipeline. The goal was to understand how security events are generated, collected, analyzed, and acted upon — mirroring the core workflow of a real SOC analyst.

The environment consists of three AWS EC2 instances: a Windows Server Domain Controller, a Windows Server client machine joined to the domain, and an Ubuntu server running Splunk. All Windows event logs are forwarded to Splunk via the Universal Forwarder, where custom SPL detection rules trigger alerts. Each alert is sent via webhook to a Tines automation workflow, where an AI agent analyzes the event and delivers a formatted SOC response to a Discord channel.

---

## Architecture

<img width="2423" height="1211" alt="image" src="https://github.com/user-attachments/assets/3582094b-0a4e-4925-b84c-b039bbc4941b" />


---

## Environment

| Machine | OS | Role |
|---------|----|----|
| EC2AMAZ-BF6VI1C | Windows Server 2022 | Domain Controller (DC01.local) |
| EC2AMAZ-6IKNFMV | Windows Server 2022 | Client Machine (domain joined) |
| ip-172-31-33-157 | Ubuntu Server | Splunk SIEM |

---

## Setup

### 1. Active Directory Domain Setup
- Deployed Windows Server 2022 as the Domain Controller
- Configured domain `DC01.local`
- Created Organizational Units (OUs): Users OU, Groups OU
- Created domain user accounts and assigned them to groups
- Joined the client machine to the domain

### 2. Group Policy Configuration
Applied the following GPOs to the Users OU:

| GPO | Purpose |
|-----|---------|
| Account Lockout | Lock accounts after 5 failed login attempts |
| Password Policy | Enforce password complexity and expiration |
| Disable Task Manager | Restrict standard users from killing processes |
| Restrict Control Panel | Prevent unauthorized system changes |
| Login Banner | Display unauthorized access warning on login |

### 3. Splunk SIEM Setup
- Deployed Splunk Enterprise on Ubuntu Server EC2 instance
- Installed Splunk Universal Forwarder on both Windows machines
- Configured forwarder to ship the following logs to Splunk index `dc01-ad`:

```ini
[WinEventLog://Security]
index = dc01-ad
disabled = false

[WinEventLog://Application]
index = dc01-ad
disabled = false

[WinEventLog://System]
index = dc01-ad
disabled = false

```

### 4. Detection Rules (Splunk Alerts)
Created 6 custom real-time detection rules in Splunk:

| Alert | Event Code | Description |
|-------|-----------|-------------|
| Authorized RDP Login | 4624 | Successful RDP login from trusted IP |
| Brute Force - Failed Logins | 4625 | 5+ failed login attempts within 5 minutes |
| New User Account Created | 4720 | New domain user account created |
| Unauthorized Successful Login | 4624 | Successful login from unknown external IP |
| User Account Disabled | 4725 | User account disabled by administrator |
| User Account Modified or Deleted | 4726/4738 | User account deleted or modified |

### 5. Tines Automation Pipeline
Configured a 3-step Tines workflow:

1. **Triggered Alerts** — Webhook receives alert payload from Splunk
2. **SOC Analyst AI Agent** — AI analyzes the alert using a custom SOC prompt
3. **Send Alerts to Discord** — Formatted analysis delivered to Discord channel

### 6. AI Agent Prompt
The AI agent was given a custom system prompt instructing it to act as a SOC analyst, respond in a consistent format, apply severity guidelines, classify IP addresses, identify false positives, and follow escalation rules based on the environment context.


---

## Alert Examples
### Brute Force - Failed Logins 

<img width="2435" height="382" alt="image" src="https://github.com/user-attachments/assets/87015ccd-6cc4-451f-9165-0b09f8885ba3" />


### Authorized RDP Login 

<img width="2636" height="369" alt="image" src="https://github.com/user-attachments/assets/6cccf09f-10a5-4af2-84ad-b446224d4a73" />

  
### New User Account Created 

<img width="2827" height="625" alt="image" src="https://github.com/user-attachments/assets/a6ac3e7e-8f1d-4d16-bf65-740a2b8f4e9f" />

  
### Unauthorized Successful Login 

<img width="2818" height="691" alt="image" src="https://github.com/user-attachments/assets/9561f93b-2c81-4431-a01f-1549441fac40" />

---

## Screenshots

> <img width="3832" height="1470" alt="image" src="https://github.com/user-attachments/assets/2d14882c-e024-463a-8334-22a6b432aba2" />

> <img width="3840" height="2088" alt="image" src="https://github.com/user-attachments/assets/8f8d923e-c40d-4413-b26d-60b4db3e44e7" />

> <img width="3863" height="2088" alt="image" src="https://github.com/user-attachments/assets/cc00c914-e454-40c9-95d2-da8d3b289b58" />

> <img width="3833" height="748" alt="image" src="https://github.com/user-attachments/assets/67e065af-981e-49f5-82cc-252374581f89" />

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ef6d9452-dd5e-4fd0-a2c2-32d063d756cc" />

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9c27707-8c4a-44e5-b460-589bde10c40a" />

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/87cd5609-b5c7-4807-8c2a-782e8d2bfac7" />


---

## Key Takeaways

- **Built a full SOC pipeline from scratch** — every layer from log collection to real-time alerting was configured manually, nothing was pre-built
- **Wrote real detection rules** — custom SPL queries targeting specific Windows event codes that map to actual attacker techniques
- **Automated alert triage with AI** — instead of just sending raw log data to Discord, an AI agent analyzes each alert and returns a formatted SOC response with severity, summary, and action steps
- **Learned to tune detection rules** — worked through noisy alerts caused by machine accounts, internal IPs, and normal system behavior to get clean and accurate detections
- **Hands-on Active Directory administration** — built the domain, created users and groups, configured OUs, and enforced security policies through GPOs

---

## Tools & Technologies

| Tool | Purpose |
|------|---------|
| AWS EC2 | Cloud infrastructure |
| Windows Server 2022 | Domain Controller and client machine |
| Ubuntu Server | Splunk host OS |
| Active Directory | Identity and access management |
| Splunk Enterprise | SIEM — log ingestion, detection, alerting |
| Splunk Universal Forwarder | Log shipping from Windows to Splunk |
| Tines | Security automation and orchestration |
| Discord | Real-time alert delivery |


---

## Disclaimer

This lab was built in an isolated AWS environment for educational purposes. All attack simulations were performed against systems I own and control. No real users or production systems were involved.
