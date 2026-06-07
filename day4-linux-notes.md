# Day 4 — Linux for Hackers

**Date:** June 2026  
**Focus:** Linux file permissions, critical system files, recon commands, Python automation

---

## File Permissions

Linux permissions control who can read, write, or execute a file.

### Reading permissions
```
-rwxr-xr-- 1 mani mani 1234 file.py
```

| Part | Meaning |
|---|---|
| `-` | File type (- = file, d = directory) |
| `rwx` | Owner: read, write, execute |
| `r-x` | Group: read and execute only |
| `r--` | Others: read only |

### Octal (number) system — interviews always test this
```
r = 4,  w = 2,  x = 1

chmod 777  →  rwxrwxrwx  (everyone full access — dangerous)
chmod 755  →  rwxr-xr-x  (owner full, others read+execute)
chmod 644  →  rw-r--r--  (owner read+write, others read only)
chmod 600  →  rw-------  (owner only — use for SSH keys)
```

### SUID bit — most important for privilege escalation
- SUID = Set User ID
- File runs with the **owner's** permissions, not the executor's
- If root owns a SUID binary and it's misconfigured → any user can become root

**Find all SUID binaries (run this on every Linux engagement):**
```bash
find / -perm -4000 -type f 2>/dev/null
```

---

## The Hacker's First 5 Commands on Any Linux System

```bash
whoami                                    # who am I?
id                                        # what groups do I belong to?
cat /etc/passwd                           # who else is on this system?
uname -a                                  # OS and kernel version
find / -perm -4000 -type f 2>/dev/null   # any exploitable SUID binaries?
```

---

## Critical Linux Files

| File | What it contains | Why attackers care |
|---|---|---|
| `/etc/passwd` | All user accounts | Maps usernames to UIDs |
| `/etc/shadow` | Password hashes | Should be root-only — if readable, hashes can be cracked |
| `/etc/sudoers` | Who can run sudo | Finding sudo access = potential root |
| `/etc/crontab` | Scheduled tasks | Writable cron = persistent backdoor |
| `~/.ssh/id_rsa` | SSH private key | Steal this = instant remote access |

---

## Essential Recon Commands

```bash
# User info
whoami
id
cat /etc/passwd

# System info
hostname
uname -a

# Network
ip a
ifconfig
ss -tulnp          # open ports
netstat -tulnp     # same, older command

# Processes
ps aux
top

# File search
find / -name "*.conf" 2>/dev/null         # config files
grep -r "password" /etc/ 2>/dev/null      # password strings
find / -perm -4000 -type f 2>/dev/null    # SUID binaries
find / -writable -type f 2>/dev/null      # writable files
```

---

## Python — subprocess module

`subprocess` lets Python run system commands and capture their output.

```python
import subprocess

result = subprocess.run('whoami', shell=True, capture_output=True, text=True)
print(result.stdout.strip())
```

- `shell=True` — run through the system shell
- `capture_output=True` — grab the output silently instead of printing directly
- `result.stdout` — the actual text output of the command
- `.strip()` — removes leading/trailing whitespace

---

## What I Found on My Kali VM

- **User:** mani (uid=1000)
- **Groups:** sudo, adm, wireshark, bluetooth (sudo group = can run as root)
- **Open ports:** 22 (SSH), 25 (SMTP)
- **SUID binaries:** /usr/bin/su, kismet_cap_* (all legitimate on Kali)
- **IP:** 10.43.223.191/24 (VirtualBox NAT network)

---

## Interview Questions I Can Now Answer

**Q: What does chmod 755 mean?**  
A: Owner has full access (rwx). Group and others have read and execute only (r-x).

**Q: What is a SUID binary and why is it dangerous?**  
A: It runs with the file owner's permissions. If root owns it and it's misconfigured, any user can get root access.

**Q: Which file stores password hashes in Linux?**  
A: /etc/shadow — should only be readable by root.

**Q: How do you find all SUID binaries on a system?**  
A: find / -perm -4000 -type f 2>/dev/null

**Q: What does the id command show?**  
A: Current user's UID, GID, and all groups they belong to.

---

## Scripts Built Today

- `linux_recon.py` — automated Linux recon covering user, ports, SUID, cron

---

*Day 4 complete. Concepts covered: file permissions, SUID, critical files, recon commands, subprocess in Python.*
