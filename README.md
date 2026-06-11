# Home Security Lab

A virtualized home lab built to simulate a real-world server environment, practice Linux administration, and demonstrate core cybersecurity concepts including firewall configuration, intrusion detection, and attack/defense simulation.

## Overview

This lab consists of two virtual machines running on VirtualBox:

- **dc-lab-01** — Ubuntu Server 24 acting as the target/victim server
- **Kali Linux** — acting as the attack machine

Both VMs are networked on a private host-only network (`192.168.56.0/24`) so they can communicate with each other and the host machine, while remaining isolated from the outside world.

## Architecture

<img width="1430" height="1040" alt="image" src="https://github.com/user-attachments/assets/66b57168-814b-405c-8c63-6e5787ca5e6f" />


Each VM also has a NAT adapter (`10.0.2.15`) for internet access, used during package installation.

## What's configured on the Ubuntu server

**nginx**
A web server serving a default HTTP page on port 80. Confirms the server is reachable from the host machine's browser at `http://192.168.56.2`.

**UFW (Uncomplicated Firewall)**
Configured with a default-deny policy. Only two ports are explicitly allowed:

- Port 22 (SSH) — remote administration
- Port 80 (HTTP) — web traffic

All other ports are silently dropped, making the server invisible to broad port scans.

**fail2ban**
Monitors `/var/log/auth.log` for failed SSH login attempts. Configured with the following rules:

```
[sshd]
enabled = true
maxretry = 3
findtime = 1m
bantime = 5m
```

Any IP that fails to authenticate 3 times within 1 minute is automatically banned at the firewall level for 5 minutes.


## Attack and defense simulation

From the Kali VM, an nmap scan was run against the Ubuntu server:

```bash
nmap -p 22,80 -v 192.168.56.2
```

Result:
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Only the two explicitly allowed ports were visible — all others were filtered.

A brute force simulation was then run by repeatedly attempting SSH login with incorrect credentials:

```bash
ssh rsiam@192.168.56.2
```

After 3 failed attempts, fail2ban detected the pattern and banned the Kali IP. This was verified on the Ubuntu server:

```bash
sudo fail2ban-client status sshd
```

The ban was also visible in real time by monitoring the auth log live:

```bash
sudo tail -f /var/log/auth.log
```

## Key concepts demonstrated

- NAT vs host-only networking in a virtualized environment
- Default-deny firewall policy and explicit port allowlisting
- The difference between filtered and closed ports
- Automated intrusion detection and IP banning
- Live log monitoring and log analysis with `grep` and `tail -f`
- Attack/defense simulation in an isolated lab environment

## Tools used

| Tool | Purpose |
|------|---------|
| VirtualBox | Virtualization platform |
| Ubuntu Server 24 | Target server OS |
| Kali Linux | Attack machine OS |
| nginx | Web server |
| UFW | Firewall management |
| fail2ban | Intrusion detection and automated banning |
| nmap | Network and port scanning |
| OpenSSH | Remote server administration |

