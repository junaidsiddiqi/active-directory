# 🏢 Active Directory Lab with Splunk SIEM & AI Automation

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

> <img width="943" height="1706" alt="image" src="https://github.com/user-attachments/assets/bccc09b0-88ab-4259-96d1-3796708bf770" />

---

## Alert Examples

### Authorized RDP Login (Low Severity)
```
✅ ALERT: Authorized RDP Login
📍 HOST: EC2AMAZ-BF6VI1C
👤 USER: Administrator
🌐 IP: [TRUSTED_IP_REDACTED] (Houston, Texas)
⚠️ SEVERITY: Low
📋 SUMMARY: Administrator successfully logged in via RDP from known trusted 
home IP. This is expected authorized activity from a known administrator source.
✅ ACTION: No action required. Log event and continue monitoring.
🕐 TIME: 2025-01-21 ~02:30 UTC
```

### New User Account Created — Suspicious (High Severity)
```
⚠️ ALERT: New User Account Created
📍 HOST: EC2AMAZ-BF6VI1C
👤 USER: hacker (created by Administrator)
🌐 IP: [Unknown]
⚠️ SEVERITY: High
📋 SUMMARY: A new user account named "hacker" was created on EC2AMAZ-BF6VI1C 
by the Administrator account. This is suspicious and requires immediate 
investigation as the account name suggests malicious intent.
✅ ACTION:
1. Immediately verify if Administrator authorized this account creation
2. Check if Administrator account has been compromised
3. Disable the "hacker" account immediately
4. Review all actions performed by this new account since creation
5. Isolate EC2AMAZ-BF6VI1C if compromise is confirmed
```

### Unauthorized Successful Login (Critical Severity)
```
🚨 ALERT: Unauthorized Successful Login
📍 HOST: EC2AMAZ-6IKNFMV
👤 USER: hacker
🌐 IP: 172.58.53.185 (Unknown - requires investigation)
⚠️ SEVERITY: Critical
📋 SUMMARY: An account named "hacker" successfully authenticated from an 
unknown external IP address. This represents a confirmed unauthorized breach 
requiring immediate containment.
✅ ACTION:
1. ISOLATE EC2AMAZ-6IKNFMV from network immediately
2. Force password reset for "hacker" account and all domain admins
3. Check for lateral movement across all hosts
4. Block 172.58.53.185 at firewall/WAF
5. Preserve all logs for forensics
6. Escalate to incident response team and AWS security
```
<details>
<summary>📸 View Screenshot</summary>

![Unauthorized Successful Login](screenshots/unauthorized_login.png)

</details>

---

## Screenshots

> <img width="3840" height="2088" alt="image" src="https://github.com/user-attachments/assets/8f8d923e-c40d-4413-b26d-60b4db3e44e7" />

> <img width="3863" height="2088" alt="image" src="https://github.com/user-attachments/assets/cc00c914-e454-40c9-95d2-da8d3b289b58" />

> <img width="3833" height="748" alt="image" src="https://github.com/user-attachments/assets/67e065af-981e-49f5-82cc-252374581f89" />

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ef6d9452-dd5e-4fd0-a2c2-32d063d756cc" />

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9c27707-8c4a-44e5-b460-589bde10c40a" />

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/87cd5609-b5c7-4807-8c2a-782e8d2bfac7" />


---

## Key Takeaways

- **End-to-end SOC pipeline** — built every layer from log collection to automated alerting
- **Detection engineering** — wrote custom SPL queries mapping to real attack techniques
- **AI automation** — integrated an AI agent that performs real-time alert triage and generates actionable SOC responses
- **False positive reduction** — refined detection rules to reduce noise from machine accounts, internal IPs, and expected system behavior
- **Active Directory administration** — configured domain, users, groups, GPOs, and security policies from scratch

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
