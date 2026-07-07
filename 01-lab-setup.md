# 01 — Lab Setup

## Objective

Before any attack or detection could happen, a controlled environment had to exist where every action is visible, isolated, and safe to run. This document explains exactly how that environment was built.

## Virtual Machines

All machines were run inside **VMware Workstation** on a single physical host, connected on the same private virtual network (`192.168.239.0/24`) so they could reach each other but stay isolated from the outside world.

| Machine | Role | IP Address | Purpose |
|---|---|---|---|
| Kali Linux | Attacker + SOC Server | `192.168.239.138` | Runs offensive tools (Nmap, Hydra) **and** hosts the Wazuh Manager, Indexer, and Dashboard |
| Ubuntu Server | Victim / Monitored Endpoint | `192.168.239.134` | Target of the attack; runs a Wazuh agent that ships its logs to the manager |
| Windows | Secondary Monitored Endpoint | *(internal lab IP)* | Added as a second Wazuh agent to demonstrate a multi-endpoint SOC view, not directly attacked in this phase |

> Running the attacker tools and the SIEM on the same Kali machine was a deliberate lab simplification — in a real SOC, the attacker and the monitoring stack would never share a host. It was done here purely to reduce the number of VMs needed while still keeping attacker and victim logically separate.

## Wazuh Installation Summary

Wazuh was installed on the Kali machine as an all-in-one stack, which includes three components working together:

1. **Wazuh Manager** — receives events from agents, applies detection rules, and generates alerts
2. **Wazuh Indexer** — stores and indexes the alert data (built on OpenSearch)
3. **Wazuh Dashboard** — the web interface used to visualize alerts, run threat hunting queries, and view MITRE ATT&CK mappings

Once installed, the dashboard was reachable at:

```
https://192.168.239.138/
```

## Agent Enrollment

The Ubuntu Server was enrolled as a Wazuh agent by:

1. Installing the `wazuh-agent` package on Ubuntu
2. Pointing the agent's configuration (`/var/ossec/etc/ossec.conf`) at the manager's IP (`192.168.239.138`)
3. Registering the agent with the manager, which assigned it **Agent ID 001** and the display name `ruansh-VMware-Virtual-Platform`
4. Starting the agent service so it began forwarding log sources — most importantly `/var/log/auth.log`, which captures every SSH login attempt

The same process was repeated for the Windows machine, which was enrolled as **Agent ID 002**.

## Verifying Connectivity

Before starting any attack, basic connectivity between Kali and the Ubuntu victim was confirmed with a simple ping test:

```bash
ping 192.168.239.134
```

This returned consistent replies with round-trip times under 2ms and 0% packet loss — confirming the victim machine was reachable and the network was healthy before recon began.

## Why This Setup Matters

A common mistake in home labs is jumping straight to "run the exploit" without documenting the environment it happened in. In a real investigation, the very first thing a SOC analyst needs is context: what's the network topology, what's being monitored, and what isn't. This lab setup section exists so anyone reading this repo — including a future version of me — can understand exactly what was in scope and how the pieces were wired together.

**Next:** [02 — Reconnaissance](02-reconnaissance.md)
