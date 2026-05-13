# Linux Fundamentals 1, 2 & 3
**Platform:** TryHackMe — Cyber Security 101 path  
**Category:** Operating Systems — Linux  
**Completed as:** Single study block (Parts 1, 2 & 3)

---

## What I learned

Linux is an open-source operating system widely used in servers, cloud infrastructure, and cybersecurity tools. Unlike Windows, its open nature allows deep customization and full transparency — which is why most security tools, web servers, and CTF environments run on Linux.

The key insight is that in Linux, **everything is a file** — configurations, logs, devices, and processes. Understanding how to navigate and manipulate those files is the foundation of both attacking and defending systems.

---

## Commands used & why they matter

| Command | Purpose |
|---|---|
| pwd | Print working directory — always know where you are |
| whoami | Shows current user — know your privilege level |
| ls | List files and folders — first recon step on any machine |
| cd | Navigate between directories |
| cat | Read file contents — inspect configs, logs, text files |
| grep | Filter specific strings — find keywords in large outputs |
| nano | Terminal text editor — create or modify files without a GUI |
| mkdir | Create directories |
| echo | Print text or write into a file |
| find | Search files by name, type, or permission — locate sensitive files |
| man | Open the manual for any command (ex: man ls) |
| wget [url] | Download files from a remote server directly into the machine |
| python3 -m http.server | Spin up a simple HTTP server to serve files to a target |
| fg | Bring a backgrounded process back to the foreground |
| ip a / ifconfig | Check network configuration and IP address |
| ssh user@ip | Connect remotely to another machine |
| rm | Delete files or directories — irreversible, no recycle bin ⚠️ |
| sudo | Run a command as root — grants elevated privileges ⚠️ |

---

## File permissions & numeric format

Every file in Linux has permissions controlling who can read, write, or execute it.

| Symbol | Meaning | Value |
|---|---|---|
| r | Read | 4 |
| w | Write | 2 |
| x | Execute | 1 |
| - | No permission | 0 |

| Symbolic | Numeric | Meaning |
|---|---|---|
| rwxrwxrwx | 777 | Everyone can do everything — dangerous |
| rwxr-xr-x | 755 | Owner full access; others read and execute |
| rw-r--r-- | 644 | Owner read/write; others read only |
| rwx------ | 700 | Only owner has any access |

> Permissions are changed with `chmod` and ownership with `chown`.  
> A file with 777 permissions in a corporate environment is a red flag.

---

## Important system directories

| Path | Purpose | Security relevance |
|---|---|---|
| /etc/passwd | All user accounts | Readable by everyone — recon target |
| /etc/shadow | Hashed passwords | Root-only — cracking = account access |
| /root | Root user home directory | Full compromise if accessible |
| /home | Regular user directories | May contain SSH keys and scripts |
| /var/log/auth.log | Authentication log | Key for incident response |

---

## Real-world application

**Evidence:** A misconfigured Linux server may expose sensitive files with weak permissions, allow SSH with weak credentials, or have users with unnecessary sudo access.

**Risk level:** High

**Impact:** An attacker with shell access can read /etc/passwd, attempt to crack /etc/shadow hashes, escalate privileges via sudo misconfigurations, serve malicious files using python3 -m http.server, and move laterally across the network.

**Mitigation:**
- Audit permissions with `find / -perm -777` and fix with `chmod`
- Restrict sudo — only grant it to users who genuinely need it
- Disable root SSH login; enforce key-based authentication
- Monitor /var/log/auth.log for suspicious activity
- Keep the system updated to patch known vulnerabilities

---

## Portfolio connections
- Google Cybersecurity Certificate
- VirtualBox Lab (Kali Linux attacking Ubuntu)
- AZ-900

SSH, file navigation, and python3 -m http.server are directly applied in the VirtualBox lab — connecting from Kali to Ubuntu uses exactly the commands covered in these rooms.
