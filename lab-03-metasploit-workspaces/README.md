# Lab 3 — Reconnaissance with Metasploit Workspaces & db_nmap

## Objective
Perform the **reconnaissance (enumeration)** stage of a penetration test: scan a target machine to discover its open ports and running service versions, then use the Metasploit **database and workspaces** to store, organize, query, and export the results. This is the foundation every later exploitation lab builds on — you can't attack a service until you know it's there, and you can't deliver a professional assessment until your findings are organized.

## What I built (lab environment)
This lab required standing up a small, isolated attacker/target lab network in VirtualBox:

| Role | Machine | IP |
|---|---|---|
| Attacker | Kali Linux | 192.168.10.8 |
| Target | Metasploitable 2 | 192.168.10.9 |

- Both VMs were placed on the **same VirtualBox NAT Network** so they share one subnet (`192.168.10.x`) and can communicate. (A plain "NAT" adapter isolates each VM and was *not* sufficient — this was a key troubleshooting lesson.)
- Connectivity was verified with `ping` from Kali to the target before scanning.
- **All addresses are private lab IPs (RFC 1918).** Metasploitable 2 is intentionally vulnerable and was kept isolated on the NAT Network — never bridged to a real network or the internet.

## Key concepts
- **Metasploit vs Metasploitable:** *Metasploit* (the `msfconsole` framework, built into Kali) is the attacker's toolkit; *Metasploitable 2* is the deliberately vulnerable target. Two separate machines.
- **Workspace:** a named, database-backed container for one engagement's data — like a case file. It stores not just a scan but the *whole* picture of a target: hosts, open services, confirmed vulnerabilities, intelligence notes, and captured loot. Workspaces keep multiple engagements isolated so data never mixes.
- **Enumeration:** discovering a target's open ports and the service + version behind each. Version numbers are the leads used to find known exploits.

## Walkthrough (what I ran)

### 1. Start the database and launch Metasploit
```bash
service postgresql start          # start PostgreSQL (stores all workspace data)
service postgresql status         # confirm it's active (running)
msfdb init                        # initialize the Metasploit database

msfconsole -q                     # launch Metasploit, -q = quiet (skip banner)
db_status                         # confirm: Connected to msf (postgresql)
```

### 2. Create a workspace for this target
```bash
workspace                         # list workspaces (* = current)
workspace -a meta2                # add + switch to a new workspace "meta2"
```

### 3. Scan the target (results auto-saved to the workspace)
```bash
db_nmap -sV 192.168.10.9          # -sV = detect service versions on open ports

# Deeper scans used in the extended version of this lab:
db_nmap -O 192.168.10.9                        # -O = OS detection (guess the operating system)
db_nmap -sC 192.168.10.9                       # -sC = run nmap's default safety scripts
db_nmap --script=ftp-anon 192.168.10.9         # check specifically for anonymous FTP login
```
Each scan writes its findings straight into the `meta2` workspace's database — that's the difference between `db_nmap` and plain `nmap`.

### 4. Query the stored recon like a database
Once data is in the workspace, it can be searched and filtered — this is where a workspace stops being a dump and becomes a tool.
```bash
hosts                             # the machines discovered
services                          # open ports + service versions (the payoff)

# Filtering the data:
services -p 21,22,80              # only these ports
services -S ftp                   # only services matching "ftp"
services -u                       # only services that are up
services -c port,proto,name,info  # choose which columns to show
services -R                       # set every listed host/service as the RHOSTS target
```
**`hosts` vs `services`:** `hosts` answers *"what machines are on the network?"* (one row per IP); `services` answers *"what's running on them?"* (one row per open port). Hosts is the building, services are the doors and windows.

### 5. The other data types a workspace stores
Hosts and services aren't the whole case file. These three commands hold the rest:
```bash
vulns                             # vulnerabilities matched to hosts (often with a CVE)
notes                             # misc intelligence auto-collected (OS fingerprints, SMB/FTP banners, HTTP page titles)
notes -t smb.fingerprint          # filter notes by type
loot                              # captured files/credentials (empty until post-exploitation)
```
- **`vulns`** — the running list of confirmed weak spots, the bridge from "what's running" to "what I can break."
- **`notes`** — the often-overlooked intelligence folder: banners, OS guesses, HTTP titles, SMB details auto-filed by type.
- **`loot`** — the trophies (dumped hashes, stolen files). Empty here, and that's the point — it fills up *after* you break in (Lab 4 onward).

### 6. Organize multiple engagements
```bash
workspace                         # list all
workspace -a target2              # add another workspace
workspace meta2                   # switch back by name
workspace -r meta2 metasploitable2 # rename
workspace -d target2              # delete
```
Switching to an empty workspace and running `hosts` returns nothing — proving each workspace's data is fully isolated.

### 7. Export a report
```bash
db_export -f xml /root/meta2_recon.xml   # dump the whole workspace to an XML file
# -f pwdump would export only captured credentials
cat /root/meta2_recon.xml | head -40      # inspect the structured output
```
This is the pentester's real deliverable — a clean, portable report of everything in the workspace, which can also be re-imported later with `db_import`.

## Results — services discovered on Metasploitable 2
The scan revealed a long list of intentionally outdated, vulnerable services, including:

| Port | Service | Version | Why it matters |
|---|---|---|---|
| 21 | ftp | vsftpd 2.3.4 | Famous backdoored version → instant root shell (CVE-2011-2523) |
| 22 | ssh | OpenSSH 4.7p1 | Old; weak-credential target |
| 23 | telnet | Linux telnetd | Credentials sent in plaintext |
| 80 | http | Apache 2.2.8 | Hosts vulnerable web apps (DVWA, Mutillidae, WordPress) |
| 139/445 | netbios/smb | Samba 3.x | Known remote code execution exploit |
| 3306 | mysql | MySQL 5.0.51a | Weak/blank credentials → database access |

(Exact list depends on the scan; `services` output captured in screenshots.)

## Analysis — reading the results like an attacker
The reconnaissance → research → exploit chain:
1. **Enumerate** — scan for open ports and service versions (this lab).
2. **Research** — take a version (e.g. `vsftpd 2.3.4`) and search Exploit-DB / `searchsploit` for a known vulnerability.
3. **Exploit** — use the matching exploit as the entry point.

**First move on this target:** `vsftpd 2.3.4` on port 21 is the obvious opening — that specific version shipped with a backdoor that grants a root shell, making it a fast, reliable way in. Samba 3.x on 445 is a strong second option.

## Key takeaways
- A shared **NAT Network** (not plain NAT) is what lets lab VMs see each other — the single most important networking detail for every attack lab.
- `db_nmap` runs the same scan as nmap but **saves results into the Metasploit database**, so workspaces can store and recall them. Flags like `-sV`, `-O`, `-sC`, and `--script=` control how deep the scan goes.
- A **workspace is a full case file**, not just a scan: `hosts`, `services`, `vulns`, `notes`, and `loot` each hold a different layer of an engagement.
- Stored recon is **queryable** — `services -p / -S / -u / -c / -R` turn the database into a working tool, including setting targets for the next attack.
- Multiple **workspaces keep engagements isolated** — essential discipline once you have more than one target.
- `db_export` turns organized recon into a **portable report** — the pentester's actual deliverable.
- Service **version numbers** are the bridge from "what's running" to "how do I exploit it." Enumeration is the first and most fundamental offensive-security skill.

## Screenshots

