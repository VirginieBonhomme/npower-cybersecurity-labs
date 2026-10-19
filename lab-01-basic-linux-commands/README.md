# Lab 1 — Basic Linux Commands (Kali)

## Objective
Get comfortable navigating a Linux system from the command line using Kali Linux. This lab covers the core terminal commands used for moving around the filesystem, creating and editing files, managing privileges, checking network information, and reviewing command history — the everyday foundation for all later cybersecurity work.

## Environment
- **OS:** Kali Linux (running as a VirtualBox VM)
- **Login:** default user `kali` / password `kali`
- **Network:** VirtualBox NAT Network — all addresses shown are **private lab IPs (RFC 1918)** and are not reachable from the public internet.

## Commands covered

| Command | What it does |
|---|---|
| `pwd` | **Print working directory** — shows the folder you're currently in (e.g. `/home/kali`). |
| `ls` | **List** files and directories in the current folder. `ls -a` also shows hidden (dot) files; `ls -l` gives a detailed long view; `ls -la` combines both. |
| `mkdir <name>` | **Make directory** — creates a new folder. |
| `cd <name>` | **Change directory** — moves into a folder. `cd ..` goes up one level; `cd ~` or `cd` returns home. |
| `echo <text>` | Prints text to the screen. With `>` it writes the text into a file (`echo Hello > one.txt`); `>>` appends instead of overwriting. |
| `nano <file>` | Opens the **nano** text editor to view/edit (or create) a file. Save & exit: `Ctrl+X` → `Y` → `Enter`. |
| `cat <file>` | Dumps a file's contents to the screen — a quick peek without opening an editor. |
| `sudo su` | Switches to the **root** (admin) user. Prompts for the password (typed invisibly). Prompt changes to `root@kali #`. Leave root with `exit`. |
| `ifconfig` | Shows network interfaces and IP addresses (Linux equivalent of `ipconfig`). The `inet` line under `eth0` is the VM's IP; `lo` is the loopback (`127.0.0.1`). |
| `rm <file>` | **Remove** a file (permanent — no recycle bin). `rm -r <dir>` removes a directory and its contents. `rm -rf` forces removal with no prompts — dangerous, use with extreme care. |
| `history` | Lists all commands run this session. `!<n>` re-runs command number n; the ↑ arrow cycles through recent commands. Stored in `~/.bash_history`. |

## Walkthrough (what I ran)
```bash
pwd                         # /home/kali
ls                          # listed default home folders
ls -la                      # viewed hidden files (.bashrc, .config, ., ..)
mkdir NPower                # created a working folder
cd NPower && pwd            # /home/kali/NPower
echo Hello World! > one.txt # wrote text into a new file
cat one.txt                 # confirmed the contents
nano one.txt                # edited and saved the file
sudo su                     # became root (prompt -> root@kali #)
exit                        # returned to the kali user
ifconfig                    # checked eth0 IP (private 10.0.2.x lab address)
rm one.txt                  # deleted the file
cd .. && rm -r NPower       # deleted the directory
history                     # reviewed the session's commands
```

## Key takeaways
- The terminal is **silent on success** — no output usually means the command worked. It only speaks up on errors.
- Linux is **case-sensitive**: `NPower`, `npower`, and `NPOWER` are three different names.
- **Flags** (like `-a`, `-l`, `-r`, `-f`) modify how a command behaves and can be stacked (`-la`).
- **Root is powerful and unforgiving** — use it only when needed, and prefer `sudo` on individual commands over living as root.
- **`rm` is permanent.** Always double-check the path before deleting, and treat `rm -rf` with real caution.

## Networking notes (from this lab)
- **Private IP ranges** (`10.x.x.x`, `172.16–31.x.x`, `192.168.x.x`) are reserved for internal/local networks and are not routable on the public internet — which is why the same addresses can exist in millions of labs at once.
- **Gateway** — the router's IP, the "exit door" to other networks (often `.1` or `.2`).
- **`127.0.0.1` (localhost)** — always refers to *this machine itself*; traffic loops back internally and never hits the network.
- Two devices can talk directly only if they share the same network portion of their IP (on a `/24` mask, the first three octets must match) — important later for making sure Kali and target VMs can reach each other.

## Screenshots
_Add your terminal screenshots here (e.g. `screenshots/pwd-ls.png`). All addresses shown are private lab IPs, safe to display._
