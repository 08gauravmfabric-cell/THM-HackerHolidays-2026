# 🐚 Hollow Shell — TryHackMe Hacker Holidays 2026

> **Hacker Holidays 2026 event**
> Category: Web + Boot2Root | Difficulty: Medium

---

## 📋 Room Summary

| Field | Details |
|---|---|
| **Room Name** | Hollow Shell |
| **Event** | TryHackMe Hacker Holidays 2026 |
| **Difficulty** | Medium |
| **Type** | Web Exploitation + Boot2Root |
| **Target** | Byte Lotus Resort Platform |
| **Skills** | Web Exploitation, Reverse Shell, Privilege Escalation |
| **User Flag** | ✅ Captured |
| **Root Flag** | ✅ Captured |

---

## 🗺️ Attack Chain Overview

```
Nmap Scan
    └─► Open Ports Identified
            └─► Web Enumeration
                    └─► Vulnerability Discovery
                            └─► Initial Foothold
                                    └─► Reverse Shell
                                            └─► User Flag ✅
                                                    └─► Enumeration
                                                            └─► Privilege Escalation
                                                                    └─► Root Flag ✅
```

---

## 📖 Room Narrative

The room references a "shell on the beach that answers back" — hinting at remote code execution and a reverse shell. The Byte Lotus beach environment has a service that accepts more than it should, providing an entry point for attackers.

---

## 🔍 Step-by-Step Walkthrough

### Step 1: Reconnaissance — Nmap

```bash
nmap -T4 -F -Pn MACHINE_IP
```

**Result:**
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
# Add any additional ports found
```

---

### Step 2: Directory Enumeration — Gobuster

```bash
gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/dirb/common.txt
```

**Found endpoints:**

| Endpoint | Status | Notes |
|---|---|---|
| `/login` | 200 | Login page |
| *(add more)* | - | - |

---

### Step 3: Web Application Analysis

```bash
# Check page source
curl http://MACHINE_IP

# Check for comments, hidden fields, credentials
curl http://MACHINE_IP/login
```

> *(Add your findings here)*

---

### Step 4: Vulnerability Discovery

> *(Document the vulnerability you found)*

**Common web vulnerabilities to check:**
- SQL Injection
- Command Injection
- File Upload bypass
- SSTI (Server Side Template Injection)
- Deserialization
- Authentication bypass

```bash
# Add your discovery commands here
```

---

### Step 5: Exploitation

> *(Document your exploitation steps)*

```bash
# Add your exploitation payload here
```

---

### Step 6: Reverse Shell

```bash
# Terminal 1 — Start listener
nc -lvnp 4444

# Terminal 2 — Send payload
# Add your reverse shell command here
```

**Shell received:**
```
connect to [YOUR_IP] from (UNKNOWN) [MACHINE_IP]
sh: 0: can't access tty; job control turned off
$
```

**Upgrade shell:**
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

---

### Step 7: User Flag

```bash
find / -name "user.txt" 2>/dev/null
cat /home/*/user.txt
```

```
THM{...}
```

---

### Step 8: Post-Exploitation Enumeration

```bash
# Who are we?
id
whoami

# Sudo permissions
sudo -l

# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Running processes
ps aux

# Cron jobs
cat /etc/crontab
ls /etc/cron.d/

# Capabilities
getcap -r / 2>/dev/null

# Open ports
ss -tlnp

# Writable files
find / -writable -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null

# Environment variables
env

# Check /etc/passwd for users
cat /etc/passwd | grep -v nologin | grep -v false
```

---

### Step 9: Privilege Escalation

> *(Document your privilege escalation path here)*

**Common privesc vectors:**
- Sudo misconfigurations
- SUID binaries (GTFOBins)
- Writable cron scripts
- Capabilities
- Password reuse
- Service running as root
- Weak file permissions

```bash
# Add your privilege escalation commands here
```

---

### Step 10: Root Flag

```bash
cat /root/root.txt
```

```
THM{...}
```

Root flag captured! 🎉

---

## 🧠 Vulnerabilities Summary

| # | Vulnerability | Severity | Location |
|---|---|---|---|
| 1 | *(Add vulnerability)* | Critical | *(Location)* |
| 2 | *(Add vulnerability)* | High | *(Location)* |
| 3 | *(Add vulnerability)* | High | *(Location)* |

---

## 🛡️ Lessons Learned

1. *(Add lesson learned)*
2. *(Add lesson learned)*
3. *(Add lesson learned)*
4. *(Add lesson learned)*
5. *(Add lesson learned)*

---

## 🔧 Tools Used

| Tool | Purpose |
|---|---|
| `nmap` | Port scanning and service detection |
| `gobuster` | Directory and endpoint enumeration |
| `curl` | HTTP requests and web analysis |
| `nc` (netcat) | Reverse shell listener |
| `python3` | Shell upgrade (pty.spawn) |
| `find` | File and SUID enumeration |
| `linpeas` | Automated privilege escalation enumeration |

---

## 📸 Screenshots

> *(Add your screenshots here)*

| Screenshot | Description |
|---|---|
| `nmap_scan.png` | Nmap results |
| `gobuster.png` | Directory enumeration results |
| `vulnerability.png` | Vulnerability discovery |
| `exploit.png` | Exploitation payload |
| `shell.png` | Reverse shell received |
| `user_flag.png` | User flag captured |
| `enumeration.png` | Post-exploitation enumeration |
| `privesc.png` | Privilege escalation |
| `root_flag.png` | Root flag captured |

---

## 📁 Files

| File | Description |
|---|---|
| `README_HollowShell.md` | This writeup |

---

## 🔗 References

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [GTFOBins — SUID/Sudo Exploitation](https://gtfobins.github.io)
- [HackTricks — Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
- [RevShells — Reverse Shell Generator](https://www.revshells.com)
- [LinPEAS — Privilege Escalation Script](https://github.com/carlospolop/PEASS-ng)

---

*Writeup by Gaurav | TryHackMe Hacker Holidays 2026*
