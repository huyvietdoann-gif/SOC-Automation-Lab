# 🔐 SOC Automation Lab – Splunk SIEM + Shuffle SOAR + TheHive Case Management

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Splunk%209.x-black)
![SOAR](https://img.shields.io/badge/SOAR-Shuffle-orange)
![Case Management](https://img.shields.io/badge/Case%20Management-TheHive%205-blue)
![Platform](https://img.shields.io/badge/Platform-VMware%20%2F%20VirtualBox-lightgrey)

> A hands-on SOC Automation lab that integrates Splunk as the SIEM, Shuffle as the SOAR engine, and TheHive for case management — simulating a real-world automated incident response pipeline.

---

## 📌 Project Overview

This project builds a SOC automation environment designed for hands-on practice and portfolio demonstration. The system integrates **Splunk** to collect and analyze logs from multiple endpoints, **Shuffle** to automate alert triage and IOC enrichment, and **TheHive** to manage security incidents end-to-end.

### Objectives

- Deploy and configure Splunk as a centralized SIEM
- Ingest logs from Windows and Linux endpoints via Splunk Universal Forwarder
- Create detection alert rules in Splunk mapped to MITRE ATT&CK techniques
- Configure Splunk webhook to trigger Shuffle workflows automatically
- Build Shuffle workflows for IOC enrichment (VirusTotal) and automated case creation
- Manage and investigate incidents in TheHive with full case lifecycle

### Technologies Used

| Component | Tool | Version |
|---|---|---|
| SIEM | Splunk Enterprise | 9.x |
| Log Forwarder | Splunk Universal Forwarder | 9.x |
| SOAR | Shuffle | Latest |
| Case Management | TheHive | 5.x |
| IOC Enrichment | VirusTotal API | v3 |
| Framework | MITRE ATT&CK | v14 |

---

## 🖧 Lab Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    SOC Analyst View                     │
│              TheHive Case Management                    │
│                  http://10.0.0.5:9000                   │
└──────────────────────────┬──────────────────────────────┘
                           │ cases + alerts
┌──────────────────────────▼──────────────────────────────┐
│                  Shuffle SOAR Engine                    │
│               http://10.0.0.5:3001                      │
│   [Enrich IOC] → [Classify Severity] → [Create Case]   │
└──────────────────────────┬──────────────────────────────┘
                           │ webhook trigger
┌──────────────────────────▼──────────────────────────────┐
│                   Splunk Enterprise                     │
│                   Ubuntu Server                         │
│                   IP: 10.0.0.5                          │
│              http://10.0.0.5:8000                       │
└───┬─────────────────────────────────────────────┬───────┘
    │ logs (Splunk UF)                             │ logs (Splunk UF)
┌───▼──────────────────┐               ┌──────────▼───────────┐
│    Windows 10        │               │      Ubuntu          │
│    IP: 10.0.0.7      │               │    IP: 10.0.0.4      │
│    - Sysmon          │               │    - Auditd          │
│    - Splunk UF       │               │    - Splunk UF       │
└──────────▲───────────┘               └──────────▲───────────┘
           │  attack (simulated)                   │
           └──────────────────┬────────────────────┘
                     ┌────────┴────────┐
                     │   Kali Linux    │
                     │  IP: 10.0.0.x   │
                     │   (Attacker)    │
                     └─────────────────┘

Network: 10.0.0.0/24 (Internal Lab Network)
```

### VM Specifications

| VM | OS | IP | RAM | CPU | Role |
|---|---|---|---|---|---|
| Splunk + Shuffle + TheHive | Ubuntu 22.04 LTS | 10.0.0.5 | 8GB | 4 core | SIEM + SOAR + Case Mgmt |
| Windows Agent | Windows 10 | 10.0.0.7 | 4GB | 2 core | Endpoint (Sysmon + Splunk UF) |
| Linux Agent | Ubuntu 20.04 | 10.0.0.4 | 2GB | 2 core | Endpoint (Auditd + Splunk UF) |
| Kali Attacker | Kali Linux | 10.0.0.x | 4GB | 2 core | Attack simulation |

---

## 📂 Documentation

| Phase | Description | Status |
|---|---|---|
| [Phase 1 – Infrastructure](docs/phase1-infrastructure.md) | Deploy Splunk, Shuffle, TheHive on Ubuntu | ✅ Completed |
| [Phase 2 – Splunk Configuration](docs/phase2-splunk-config.md) | Install Splunk UF, ingest Windows & Linux logs | ✅ Completed |
| [Phase 3 – Alert Rules](docs/phase3-alert-rules.md) | Create detection rules + webhook trigger in Splunk | ✅ Completed |
| [Phase 4 – Shuffle Workflow](docs/phase4-shuffle-workflow.md) | Build SOAR automation workflow with IOC enrichment | ✅ Completed |
| [Phase 5 – TheHive Integration](docs/phase5-thehive-integration.md) | Case management, investigation, escalation | ✅ Completed |

---

## 📁 Repository Structure

```
SOC-Automation-Splunk-SOAR/
├── README.md
├── docs/
│   ├── phase1-infrastructure.md
│   ├── phase2-splunk-config.md
│   ├── phase3-alert-rules.md
│   ├── phase4-shuffle-workflow.md
│   └── phase5-thehive-integration.md
├── configs/
│   ├── inputs.conf               # Splunk UF input config
│   ├── outputs.conf              # Splunk UF output config
│   ├── splunk-alerts.conf        # Saved search / alert definitions
│   └── shuffle-workflow.json     # Exported Shuffle workflow
└── screenshots/
    ├── phase1/
    ├── phase2/
    ├── phase3/
    ├── phase4/
    └── phase5/
```

---

## 🔄 Automation Flow

```
[Splunk detects alert]
        ↓
[Webhook → Shuffle triggered]
        ↓
[Extract: src_ip, username, rule_name]
        ↓
[VirusTotal API → enrich src_ip]
        ↓
[Classify severity: Low / Medium / High]
        ↓
[Create Case in TheHive]
        ↓
[Assign task → SOC Analyst investigates]
```

---

## 🎯 MITRE ATT&CK Coverage

| Tactic | Technique | Splunk Alert | Shuffle Action | TheHive Case |
|---|---|---|---|---|
| Credential Access | T1110.001 – Brute Force | ✅ | ✅ | ✅ |
| Execution | T1059.001 – PowerShell | ✅ | ✅ | ✅ |
| Defense Evasion | T1070.004 – File Deletion | ✅ | ✅ | ✅ |
| Lateral Movement | T1021.001 – RDP | ✅ | ✅ | ✅ |
| Persistence | T1053.005 – Scheduled Task | ✅ | ✅ | ✅ |