# 02 — Reconnaissance

## Objective

Before attacking anything, an attacker needs to know what's actually there: which ports are open, what services are running, and what operating system is in play. This phase used **Nmap**, the industry-standard network scanning tool, to build that picture of the Ubuntu victim machine.

Every real intrusion starts here — this is the "casing the building" phase before anyone tries a door handle.

## Step 1: Confirm the Host Is Alive

```bash
ping 192.168.239.134
```

**Result:** 9/9 packets received, 0% packet loss, average round-trip time ~1.6ms. This confirms the target is up and reachable on the network before spending time scanning it.

## Step 2: Default Port Scan

```bash
nmap 192.168.239.134
```

Nmap's default scan checks the 1,000 most common TCP ports.

**Result:**
| Port | State | Service |
|---|---|---|
| 22/tcp | open | ssh |
| 80/tcp | open | http |

The MAC address was also captured (`00:0C:29:6F:8D:5E`), confirming this is a VMware virtual machine — useful for sanity-checking the lab environment itself.

## Step 3: Full Port Range Scan

```bash
nmap -p- 192.168.239.134
```

The default scan only checks common ports — to be thorough, a full scan of all 65,535 TCP ports was run. This is important because attackers (and analysts auditing their own systems) shouldn't assume a service is only listening where it's "supposed to."

**Result:** Same two ports confirmed open (22, 80) out of the full range — no hidden or unusual services were found.

## Step 4: Service and Version Detection

```bash
nmap -sV -p 22 192.168.239.134
```

**Result:** `OpenSSH 9.6p1 Ubuntu 3ubuntu13.16`, running protocol 2.0.

Knowing the exact SSH version matters — it tells an attacker (or defender doing a vulnerability assessment) whether known CVEs apply to that specific version.

## Step 5: OS Fingerprinting

```bash
nmap -O 192.168.239.134
```

**Result:** Device type identified as general purpose, running **Linux kernel 4.15–5.19**, with a network distance of 1 hop (confirming it's on the same local segment as Kali).

## Step 6: Aggressive Scan (All-in-One)

```bash
nmap -A -p- 192.168.239.134
```

This combines OS detection, version detection, script scanning, and traceroute into a single comprehensive scan.

**Result:** Everything from the previous steps was confirmed in one pass, plus:
- **Apache httpd 2.4.58** on port 80, serving the default "Apache2 Ubuntu Default Page"
- SSH host keys fingerprinted (ECDSA and ED25519)
- Traceroute confirmed a single hop between Kali and the Ubuntu victim

## Step 7: SSH-Specific Script Scans

Two targeted NSE (Nmap Scripting Engine) scripts were run to understand the SSH service in more depth:

```bash
nmap --script ssh2-enum-algos 192.168.239.134
```

This enumerated every key exchange, host key, encryption, and MAC algorithm the SSH server supports — useful for assessing whether weak/legacy algorithms are still enabled (none were found here; the server supports modern algorithms like `curve25519-sha256` and `chacha20-poly1305`).

```bash
nmap --script ssh-auth-methods 192.168.239.134
```

**Result:** The server accepts two authentication methods: **publickey** and **password**. This single finding is what made the next phase possible — since password authentication is enabled, a brute-force attack becomes a viable attack path.

## Summary of Findings

| Category | Finding |
|---|---|
| Open ports | 22 (SSH), 80 (HTTP) |
| SSH version | OpenSSH 9.6p1 (Ubuntu) |
| Web server | Apache 2.4.58 |
| OS | Linux kernel 4.15 – 5.19 |
| Auth methods on SSH | publickey, **password** ⚠️ |

The fact that password authentication was enabled on SSH is the single most important finding from this phase — it directly shaped the exploitation strategy in the next stage.

**Screenshots:** see `images/recon/`

**Next:** [03 — Exploitation](03-exploitation.md)
