# 04 — Detection & Response

## Objective

This is the phase that turns this project from "I ran a brute-force attack" into an actual SOC monitoring exercise. Every action from the previous two phases generated log entries on the Ubuntu victim. Those logs were shipped in real time to the Wazuh Manager by the installed agent, where they were parsed, correlated into alerts, scored by severity, and automatically mapped to the MITRE ATT&CK framework — all without any manual intervention. This document walks through what the SOC dashboard showed and how an analyst would investigate it.

## Dashboard Overview

The Wazuh **Overview** dashboard gives the first, highest-level view of the environment's health over the last 24 hours:

| Severity | Count |
|---|---|
| Critical (level 15+) | 0 |
| High (level 12–14) | 0 |
| Medium (level 7–11) | 610 |
| Low (level 0–6) | 2,048 |

One agent shown as **Active**, zero disconnected — confirming the Ubuntu endpoint stayed connected and reporting throughout the entire exercise.

## Threat Hunting: The Authentication Story

Drilling into the **Threat Hunting** module (filtered to this agent) over a 30-day window shows the real story:

| Metric | Value |
|---|---|
| Total alerts | 3,486 |
| Level 12+ alerts | 0 |
| Authentication failures | 2,632 |
| Authentication successes | 47 |

The **"Top 10 Alert Groups"** timeline chart shows a dramatic, sharp spike right at the point the Hydra attack ran — a flat baseline for weeks, then a near-vertical jump in `authentication_failed` and `sshd` alert volume. This is precisely the kind of pattern a SOC analyst is trained to notice at a glance: a sudden, sustained anomaly against an otherwise quiet baseline.

The **Top 5 Rule Groups** breakdown confirms `syslog`, `authentication_failed`, and `sshd` as the dominant categories during the incident window — exactly what you'd expect from a brute-force attempt, and nothing that points to unrelated noise.

## Drilling Into a Single Alert

Rather than stopping at the dashboard totals, a real investigation means opening an individual alert and reading what it actually says. Clicking into one of the thousands of `sshd: authentication failed` events surfaced this:

```
full_log: "2026-07-05T12:18:00.190351+02:00 ruansh-VMware-Virtual-Platform sshd[6866]: 
Failed password for rushi from 192.168.239.138 port 50690 ssh2"
```

And the structured fields extracted from it:

| Field | Value |
|---|---|
| `rule.id` | 5760 |
| `rule.level` | 5 |
| `rule.description` | sshd: authentication failed |
| `data.srcip` | 192.168.239.138 (the Kali attacker) |
| `data.dstuser` | rushi |
| `decoder.name` | sshd |
| `rule.groups` | syslog, sshd, authentication_failed |

This single log line is the digital fingerprint of one failed Hydra attempt — and Wazuh automatically parsed the raw text into structured, searchable fields (source IP, target username, port) without any manual regex work.

## MITRE ATT&CK Enrichment — The Key Finding

The most valuable part of this alert is what Wazuh added on top of the raw log:

| Field | Value |
|---|---|
| `rule.mitre.id` | T1110.001, T1021.004 |
| `rule.mitre.tactic` | Credential Access, Lateral Movement |
| `rule.mitre.technique` | Password Guessing, SSH |
| `rule.pci_dss` | 10.2.4, 10.2.5 |
| `rule.hipaa` | 164.312.b |
| `rule.gdpr` | IV_35.7.d, IV_32.2 |
| `rule.nist_800_53` | AU.14, AC.7 |

This means every single failed login wasn't just logged — it was automatically classified against a real adversary technique (**T1110.001 – Password Guessing**) under two ATT&CK tactics (**Credential Access** and **Lateral Movement**), and simultaneously mapped against four different compliance frameworks (PCI DSS, HIPAA, GDPR, NIST 800-53).

This is exactly what separates a basic log viewer from a real SIEM: the platform did the threat-intelligence correlation automatically, at the moment the alert was generated.

## The MITRE ATT&CK Dashboard View

Switching to Wazuh's dedicated **MITRE ATT&CK** module and filtering by events confirms the pattern at scale: **2,463 hits**, almost entirely tagged with technique **T1110** or **T1110.001 / T1021.004**, under the **Credential Access** and **Credential Access + Lateral Movement** tactics. Rule 2502 (`syslog: User missed the password more than one time`) fired at the higher severity level 10, marking repeated-failure thresholds being crossed — the system escalating severity as the same source kept failing against the same account.

## Escalation Pattern Noticed

Two distinct rule IDs told two chapters of the same story:

- **Rule 5760** (level 5) — a single failed password attempt. Low severity on its own; happens occasionally even from legitimate typos.
- **Rule 2502** (level 10) — fires when the *same user missed the password more than once* in a short window. This is Wazuh's correlation engine recognizing that this isn't a one-off mistake — it's a pattern, and it escalates the severity accordingly.

This escalation logic is exactly how real SOC detection rules are designed: individual events are low-noise, but the *pattern* of repetition is what triggers a higher-confidence alert.

## Vulnerability Context

As a final piece of situational awareness, the **Vulnerability Detection** dashboard for the same agent was reviewed to understand what else might be exposed on this host:

| Severity | Count |
|---|---|
| Critical | 2 |
| High | 197 |
| Medium | 555 |
| Low | 11 |

Top identified CVEs included `CVE-2024-3661`, `CVE-2013-7445`, `CVE-2015-8553`, `CVE-2016-8660`, and `CVE-2017-0537`, primarily tied to the Ubuntu 24.04.4 LTS kernel packages. This matters because it shows the full risk picture: this isn't just a host that got brute-forced — it's a host that, if the brute-force *had* succeeded, also had a meaningful number of unpatched vulnerabilities that could support post-exploitation activity.

## Analyst Conclusion (Incident Summary)

**What happened:** A sustained SSH password brute-force attack was carried out from `192.168.239.138` (Kali) against the `rushi` account on `192.168.239.134` (Ubuntu) over a period of several minutes, generating over 2,600 failed authentication attempts.

**What the SIEM did:** Wazuh correctly identified every attempt, escalated severity once repeated failures against the same account were detected, and automatically mapped the activity to MITRE ATT&CK technique T1110.001 (Password Guessing) under the Credential Access tactic.

**Was the attack successful?** No successful login from the attacking source was observed in the authentication-success counters tied to this activity — the account's password held against this wordlist run.

**Recommended response (what a SOC analyst would do next):**
1. Block or rate-limit the source IP at the firewall/fail2ban level
2. Force a password reset for the targeted account regardless of outcome
3. Enforce SSH key-based authentication only, disabling password auth entirely
4. Review whether this account should be exposed to SSH at all, or restricted by IP allow-listing

Full hardening recommendations are in the next document.

**Screenshots:** see `images/wazuh-dashboard/`

**Next:** [05 — Lessons Learned](05-lessons-learned.md)
