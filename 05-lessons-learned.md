# 05 — Lessons Learned

## What Worked Well

- **Wazuh's out-of-the-box rules caught the attack without any custom rule-writing.** Rule 5760 and Rule 2502 fired correctly on default configuration — no tuning was needed to detect a textbook SSH brute-force pattern.
- **MITRE ATT&CK mapping happened automatically.** Every alert arrived pre-tagged with the correct tactic and technique (T1110.001, Credential Access), which meant no manual threat-intelligence lookup was required to understand what kind of attack this was.
- **Compliance mapping was a nice surprise.** Seeing PCI DSS, HIPAA, GDPR, and NIST 800-53 references attached to the same alert showed how a single detection can serve multiple audiences — a security analyst, an auditor, and a compliance officer could all pull relevant information from the same event.
- **The severity escalation logic made sense.** A single failed login (level 5) is very different from repeated failures against the same account (level 10) — and the platform reflected that difference correctly instead of treating every failed login identically.

## Gaps and Limitations Identified

- **This lab only tested one attack technique (password brute-force).** It doesn't yet demonstrate detection of privilege escalation, lateral movement between the Ubuntu and Windows agents, persistence mechanisms, or malware execution — all common next stages in a real intrusion.
- **No active response was configured.** Wazuh supports active response (e.g., automatically blocking an IP after N failed attempts via `iptables`/`fail2ban` integration), which wasn't enabled here. The alerts were observed, but nothing automatically intervened.
- **Single attacker source.** Real-world brute-force traffic often comes from distributed botnets across many IPs, which is harder to detect with simple source-IP-based rules alone. This lab only tested detection against a single, obvious source.
- **The password ultimately wasn't tested to completion** against the full 13-million-entry wordlist — a real attacker (or a stronger lab test) might run this for hours or days.

## Hardening Recommendations (What I'd Change on This Server)

1. **Disable SSH password authentication entirely**, allowing only key-based auth (`PasswordAuthentication no` in `sshd_config`). This alone would have made this entire attack path impossible.
2. **Deploy fail2ban** to automatically block source IPs after a small number of failed attempts, turning detection into automatic response.
3. **Enforce MFA** on any account that must retain password authentication.
4. **Restrict SSH access by source IP** using a firewall allow-list, rather than exposing port 22 to the entire local network (or the internet, in a production scenario).
5. **Configure Wazuh Active Response** to automatically act on Rule 2502-type escalations instead of requiring manual review.

## Roadmap — Where This Lab Goes Next

This project is intentionally structured to grow. The next phases planned for this lab, following the same document-everything approach, are:

| Planned Stage | MITRE Tactic | Goal |
|---|---|---|
| Privilege Escalation | Privilege Escalation | Exploit a misconfigured sudo rule or SUID binary on the Ubuntu host and observe whether/how Wazuh's auditd integration detects it |
| Persistence | Persistence | Add a cron job or SSH authorized key as an attacker foothold, and test Wazuh's File Integrity Monitoring (FIM) detection |
| Lateral Movement | Lateral Movement | Pivot from the Ubuntu agent toward the Windows agent to test cross-endpoint visibility in Wazuh |
| Malware / C2 Simulation | Command and Control | Drop a safe, industry-standard **EICAR test file** (a benign string used to test antivirus detection, not real malware) and observe Wazuh's VirusTotal integration flag it |
| Full Incident Response Report | — | Write a complete IR report simulating what a SOC analyst would deliver to leadership after a real multi-stage intrusion |

## Key Takeaway

The most important lesson from this lab isn't the attack itself — it's that **visibility is only useful if someone actually looks at it and understands what they're seeing**. Wazuh generated thousands of alerts automatically, but the value came from walking through the dashboard, opening individual events, reading the raw logs behind them, and connecting that back to a documented framework (MITRE ATT&CK). That process — attack, detect, investigate, conclude, recommend — is the actual job of a SOC analyst, and this lab was built to practice exactly that loop.

**Back to:** [README](../README.md)
