# Lab 2 — Checking File Integrity with Hashing (MD5 / SHA-256)

## Objective
Use cryptographic hashes to verify whether a file has been modified. This lab demonstrates how a hash acts as a file's "fingerprint," how to record and later verify a checksum, how an integrity check reacts when a file is tampered with, and why MD5 has been superseded by SHA-256 for security purposes.

## Background — what is hashing?
A **hash** is a fixed-length string produced from input data by a hashing algorithm. Key properties:
- The **same input always produces the same hash.**
- **Any** change to the input — even a single character — produces a completely different hash.
- It is **one-way**: you cannot reverse a hash back into the original data.

These properties make hashing ideal for **integrity verification**: record a file's hash now, re-hash it later, and compare. Matching hashes mean the file is unchanged; different hashes mean it was altered.

## Environment
- **OS:** Kali Linux (VirtualBox VM)
- **Tools:** `md5sum`, `sha256sum` (built into Linux)

## Walkthrough (what I ran)
```bash
# 1. Create a working directory and move into it
mkdir md5dir
cd md5dir

# 2. Create a file and generate its MD5 checksum
echo "Hello World" > file1
md5sum file1
# -> e59ff97941044f85df5297e1c302d260  file1   (32-char hash + filename)

# 3. Save the checksum to a file so it can be verified later
md5sum file1 > hash
cat hash

# 4. Verify integrity — file is unchanged
md5sum --check hash
# -> file1: OK

# 5. Modify the file (append a line), then re-check
echo "Good Night" >> file1       # >> appends; > would overwrite
cat file1                        # now shows both lines
md5sum --check hash
# -> file1: FAILED
#    md5sum: WARNING: 1 computed checksum did NOT match

# 6. (Optional) SHA-256 and the lesson about mixing hash types
sha256sum file1 >> hash          # appends a 64-char SHA-256 line into the MD5 hash file
md5sum --check hash
# -> file1: FAILED
#    md5sum: hash: 2: improperly formatted MD5 checksum line
#    (md5sum cannot read the SHA-256 line — wrong tool for that hash)

# The correct way to use SHA-256 — matching tool, its own file:
sha256sum file1 > hash256
sha256sum --check hash256
# -> file1: OK
```

## What each command does
| Command | Purpose |
|---|---|
| `md5sum file1` | Computes the MD5 checksum (fingerprint) of `file1`. Output is a 32-character hex hash + the filename. |
| `md5sum file1 > hash` | Saves that checksum line into a file named `hash` for later verification. |
| `md5sum --check hash` (or `-c`) | Re-hashes the listed file and compares it to the recorded hash. Reports `OK` or `FAILED`. |
| `echo "..." >> file1` | Appends a line to the file (`>>` adds, `>` overwrites). Used here to simulate tampering. |
| `sha256sum file1` | Computes a SHA-256 checksum — a 64-character hex hash. |
| `sha256sum --check hash256` | Verifies a file against a recorded SHA-256 checksum. |

## Results & analysis
- **Before modification:** `md5sum --check hash` returned `file1: OK` — the file matched its recorded hash.
- **After appending a line:** the same check returned `file1: FAILED`. Adding even one line changed `file1`'s content, so its freshly computed MD5 no longer matched the stored value. The mismatch proves the file was modified.
- **Mixing hash types:** appending a SHA-256 hash into the MD5 `hash` file and re-running `md5sum --check` produced an *"improperly formatted MD5 checksum line"* error. **Lesson:** a hash must be verified with its matching tool — `md5sum` for MD5, `sha256sum` for SHA-256. `md5sum` only understands 32-character MD5 hashes.

## Key takeaways
- A hash is a reliable way to **detect change you can't see** — tampered configs, modified binaries, corrupted downloads.
- **`>` overwrites, `>>` appends** — an important distinction when handling files.
- Verify a checksum with the **tool that matches the algorithm**.
- **MD5 is cryptographically broken**: attackers can craft two different files with the *same* MD5 (a collision), so a matching MD5 no longer guarantees authenticity. It is still useful for catching *accidental* corruption.
- **SHA-256 is the modern standard** for security-sensitive integrity checks, with no practical collision weakness.

## Real-world relevance
When you download software, vendors often publish a checksum (increasingly SHA-256). Hashing your downloaded copy and comparing confirms the file wasn't corrupted in transit or swapped for a malicious version. The same principle underpins file-integrity monitoring tools that alert defenders when critical system files change unexpectedly.

## Screenshots
_Add your terminal screenshots here (e.g. `screenshots/md5-ok.png`, `screenshots/md5-failed.png`)._
