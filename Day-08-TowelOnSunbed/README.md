# 🏖️ Towel on Sunbed — TryHackMe Hacker Holidays 2026

> **Hacker Holidays 2026 event**
> Category: Web + Boot2Root | Difficulty: Medium

---

## 📋 Room Summary

| Field | Details |
|---|---|
| **Room Name** | Towel on Sunbed |
| **Event** | TryHackMe Hacker Holidays 2026 |
| **Difficulty** | Medium |
| **Type** | Web Exploitation + Boot2Root |
| **Target** | Byte Lotus Resort Platform |
| **User Flag** | ✅ Captured |
| **Root Flag** | ✅ Captured |

---

## 🗺️ Attack Chain Overview

```
Nmap Scan
    └─► Open Ports Identified
            └─► Web Enumeration
                    └─► Vulnerability Discovery
                            └─► Initial Access
                                    └─► User Flag ✅
                                            └─► Privilege Escalation
                                                    └─► Root Flag ✅
```

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
curl http://MACHINE_IP
```

> *(Add your findings here)*

---

### Step 4: Vulnerability Discovery

> *(Document the vulnerability you found here)*

**Payload used:**
```bash
# Add your exploitation commands here
```

---

### Step 5: Initial Access — Reverse Shell

```bash
# Start listener
nc -lvnp 4444

# Send payload
# Add your reverse shell payload here
```

**Shell received:**
```
connect to [YOUR_IP] from (UNKNOWN) [MACHINE_IP]
$ id
uid=?(user) gid=?(user) groups=?(user)
```

---

### Step 6: User Flag

```bash
find / -name "user.txt" 2>/dev/null
cat /home/*/user.txt
```

```
THM{...}
```

---

### Step 7: Post-Exploitation Enumeration

```bash
# Check sudo permissions
sudo -l

# Check SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Check running processes
ps aux

# Check cron jobs
cat /etc/crontab
ls /etc/cron.d/

# Check capabilities
getcap -r / 2>/dev/null

# Check open ports
ss -tlnp
```

---

### Step 8: Privilege Escalation

> *(Document your privesc path here)*

```bash
# Add your privilege escalation commands here
```

---

### Step 9: Root Flag

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
| `python3` | Shell upgrade |

---

## 📸 Screenshots

> *(Add your screenshots here)*

| Screenshot | Description |
|---|---|
| `nmap_scan.png` | Nmap results |
| `gobuster.png` | Directory enumeration |
| `vulnerability.png` | Vulnerability discovery |
| `shell.png` | Reverse shell received |
| `user_flag.png` | User flag captured |
| `privesc.png` | Privilege escalation |
| `root_flag.png` | Root flag captured |

---

## 📁 Files

| File | Description |
|---|---|
| `README_TowelOnSunbed.md` | This writeup |

---

## 🔗 References

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [GTFOBins](https://gtfobins.github.io)
- [HackTricks](https://book.hacktricks.xyz)

---

*Writeup by Gaurav | TryHackMe Hacker Holidays 2026*
