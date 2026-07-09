# 🛡️ SecureOps SOC Monitoring Lab

**A hands-on home lab documenting a full attack-and-detection cycle — from reconnaissance to SIEM-driven incident analysis — built to practice both offensive techniques and SOC (Security Operations Center) monitoring using Wazuh.**

---

## 📌 Project Overview

This project simulates a real-world attack scenario inside an isolated virtual lab and shows how a Security Operations Center would detect, investigate, and respond to it. It was built entirely on VMware Workstation using three machines:

- **Kali Linux** — the attacker machine, also hosting the Wazuh Manager, Indexer, and Dashboard (acting as the SOC's SIEM stack)
- **Ubuntu Server** — the victim machine, running SSH and Apache, monitored by a Wazuh agent
- **Windows** — a second monitored endpoint, added to demonstrate multi-agent visibility

The goal isn't just to "run an attack" — it's to show the **full loop**: attacker performs recon and exploitation → victim generates logs → Wazuh agent forwards those logs → Wazuh Manager correlates them into alerts → those alerts are mapped to the **MITRE ATT&CK framework** → and finally, a human (me, acting as analyst) investigates and writes up the incident.

This mirrors exactly what a junior SOC analyst or blue team engineer does day-to-day: watch a dashboard, click into an alert, understand what happened, and decide what to do about it.

---

## 🧭 Table of Contents

| Doc | What it covers |
|---|---|
| [01 – Lab Setup](docs/01-lab-setup.md) | Network topology, VM configuration, Wazuh installation & agent enrollment |
| [02 – Reconnaissance](docs/02-reconnaissance.md) | Host discovery, port scanning, service/OS fingerprinting with Nmap |
| [03 – Exploitation](docs/03-exploitation.md) | SSH brute-force attempt using Hydra |
| [04 – Detection & Response](docs/04-detection-response.md) | Wazuh dashboards, alert breakdown, MITRE ATT&CK mapping, root-cause analysis |
| [05 – Lessons Learned](docs/05-lessons-learned.md) | Hardening recommendations, detection gaps, and roadmap for expanding this lab |
| [scripts/commands.md](scripts/commands.md) | Every command used, in exact execution order, for reproducibility |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────┐
│   Kali Linux (Attacker + SOC Stack) │
│   192.168.239.138                   │
│   ├─ nmap / hydra (attacker tools)  │
│   └─ Wazuh Manager + Indexer +      │
│        Dashboard (SIEM)             │
└───────────────┬─────────────────────┘
                │ attacks (SSH recon + brute force)
                ▼
┌─────────────────────────────────────┐
│   Ubuntu Server (Victim)            │
│   192.168.239.134 — Agent ID 001    │
│   ├─ OpenSSH 9.6p1, Apache 2.4.58   │
│   └─ Wazuh Agent → forwards         │
│        /var/log/auth.log events     │
└───────────────┬─────────────────────┘
                │ encrypted agent traffic (1514/1515)
                ▼
        Wazuh Manager correlates logs
        into alerts, tags them with
        MITRE ATT&CK IDs, and surfaces
        them on the Dashboard

┌─────────────────────────────────────┐
│   Windows Host (Second Endpoint)    │
│   Agent ID 002                      │
│   Added for multi-endpoint          │
│   visibility across the SOC         │
└─────────────────────────────────────┘
```

A rendered version of this diagram is in `images/architecture-diagram.png`.

---

## 🔍 What This Lab Demonstrates

- **Offensive fundamentals:** host discovery, port/service enumeration, OS fingerprinting, and credential brute-forcing using industry-standard tools (Nmap, Hydra)
- **Defensive monitoring:** deploying and configuring a real SIEM (Wazuh) to collect, parse, and correlate logs from a remote agent
- **Threat detection mapped to MITRE ATT&CK:** alerts weren't just "noticed" — they were tied to specific tactics (Credential Access, Lateral Movement) and techniques (T1110.001 – Password Guessing, T1021.004 – SSH)
- **Analyst-level investigation:** drilling into individual alerts (`rule.id`, `full_log`, `decoder`, `rule.mitre.id`) instead of just looking at dashboard totals
- **Vulnerability awareness:** cross-referencing the same endpoint's exposed CVEs using Wazuh's Vulnerability Detection module

---

## 🖥️ Tools Used

| Tool | Purpose |
|---|---|
| VMware Workstation | Virtualization platform hosting all three machines |
| Kali Linux | Attacker OS and Wazuh server host |
| Ubuntu Server | Target/victim machine |
| Nmap | Network reconnaissance and service fingerprinting |
| Hydra | SSH credential brute-force testing |
| Wazuh (Manager, Indexer, Dashboard) | SIEM — log collection, correlation, alerting, ATT&CK mapping |
| MITRE ATT&CK Framework | Standardized mapping of adversary tactics/techniques |

---

## 📸 Key Findings at a Glance

- Nmap identified 2 open ports on the victim (22/SSH, 80/HTTP) and correctly fingerprinted the OS as Linux 4.15–5.19 running OpenSSH 9.6p1 and Apache 2.4.58.
- Hydra's brute-force run against SSH generated a sustained stream of failed authentication attempts (~70 tries/minute).
- Wazuh caught every single attempt in real time — over 2,600 authentication failures were logged and correlated under **Rule 5760 (sshd: authentication failed)** and **Rule 2502 (user missed the password more than once)**.
- Each alert was automatically tagged with **MITRE ATT&CK ID T1110.001 (Password Guessing)** under the **Credential Access** and **Lateral Movement** tactics — meaning no manual tagging was needed; the SIEM did the classification.
- The same agent's Vulnerability Detection dashboard surfaced **2 Critical, 197 High, and 555 Medium severity CVEs**, showing that detection and vulnerability context live side-by-side in one platform.

Full breakdown with screenshots: see [04 – Detection & Response](docs/04-detection-response.md).

---

## ⚠️ Disclaimer

This project was performed entirely within an isolated, self-owned virtual lab environment for educational purposes. No systems outside this lab were scanned, tested, or accessed. The techniques shown here should only ever be used against systems you own or have explicit written authorization to test.

---

## 📄 License
MIT License — free to use, modify, and distribute with attribution.

## 👤 Author

Built and documented as part of a hands-on SOC/blue-team skills portfolio.
Feel free to open an issue or reach out if you have suggestions for extending this lab (privilege escalation, lateral movement, and malware-simulation stages are planned — see [Lessons Learned](docs/05-lessons-learned.md)).
