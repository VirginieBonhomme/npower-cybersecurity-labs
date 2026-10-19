# Cybersecurity Labs — NPower

**Virginie Solomon-Bonhomme**
_Aspiring cybersecurity analyst · NPower cybersecurity program_

<!-- Add your links: [GitHub](https://github.com/your-username) · [LinkedIn](https://www.linkedin.com/in/your-profile) -->

A collection of hands-on cybersecurity labs completed as part of the **NPower** cybersecurity training program. Each lab is documented in its own folder with objectives, the exact commands/steps I ran, screenshots, and what I learned.

All labs are performed in an isolated virtual lab environment (Kali Linux + target VMs in VirtualBox). **Any IP addresses shown are private lab addresses (RFC 1918) and are not reachable from the public internet.** No real systems, credentials, or production data are involved.

## Lab environment
- **Attacker:** Kali Linux (VirtualBox VM)
- **Targets:** Metasploitable 2 / Metasploitable 3 and other intentionally vulnerable VMs (introduced in later labs)
- **Networking:** VirtualBox NAT Network, so all VMs share one private subnet

## Labs

| # | Lab | Topic | Status |
|---|-----|-------|--------|
| 1 | [Basic Linux Commands](./lab-01-basic-linux-commands/) | Linux terminal fundamentals | ✅ Complete |
| 2 | [Checking File Integrity with Hashing](./lab-02-file-integrity-hashing/) | Hashing & integrity (MD5/SHA-256) | ✅ Complete |
| 3 | [Metasploit Workspaces & Recon](./lab-03-metasploit-workspaces/) | Enumeration (nmap, workspaces) | ✅ Complete |
| 4 | [SSH Brute Force](./lab-04-ssh-brute-force/) | Credential attacks & remote access (Metasploit `ssh_login`) | ✅ Complete |
| 5 | WordPress Exploitation | Web app exploitation | ⬜ Planned |
| 6 | Bash Scripting | Automation | ⬜ Planned |
| 7 | Post-Exploitation | Metasploit modules | ⬜ Planned |
| 8 | RDP Exploitation | Remote desktop | ⬜ Planned |
| 9 | Nessus Installation | Vulnerability scanning setup | ⬜ Planned |
| 10 | Nessus Vulnerability Scan | Vulnerability scanning | ⬜ Planned |
| 11 | SQL Injection | Web app / database attacks | ⬜ Planned |
| 12 | Cross-Site Scripting (XSS) | Web app attacks | ⬜ Planned |
| 13 | nftables Firewall | Network defense | ⬜ Planned |

_(Table updates as each lab is completed.)_

## Skills demonstrated
- Linux command-line administration
- File integrity verification with cryptographic hashes
- Secure remote access (SSH)
- Vulnerability scanning (Nessus)
- Web application exploitation (SQLi, XSS, WordPress)
- Post-exploitation with Metasploit
- Network firewalling (nftables)

## About
These labs reflect practical, entry-level cybersecurity skills built through the NPower program. Each write-up is my own documentation of the work, intended to show both *what* I did and *why* it matters.

> ⚠️ **Ethics note:** All techniques here were performed in a self-contained lab against systems I own and control, for educational purposes only. Never apply these techniques to systems you do not have explicit permission to test.
