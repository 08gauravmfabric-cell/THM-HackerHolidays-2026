# 🏖️ TryHackMe — Hacker Holidays 2026
> *"Everyone's on holiday. Everyone's hiding something."*

![TryHackMe](https://shields.io)
![Event](https://shields.io)
![Rooms](https://shields.io)
![Status](https://shields.io)

---

## 🏨 About the Event

**Hacker Holidays 2026** is a free 14-day cybersecurity challenge hosted by TryHackMe, set inside the fictional **Byte Lotus Hotel** — a five-star resort with a zero-star security posture.

Starting **July 27, 2026**, a new beginner-friendly room unlocks daily at **4PM UTC**, covering:

- 🔍 OSINT
- 🌐 Web Exploitation
- ☁️ Cloud Security
- 🔬 Digital Forensics
- 🤖 AI Prompt Attacks

Every completed room earns a raffle ticket for the **$50,000+ prize pool**.

---

## 🗺️ Room Progress

| Day | Room Name | Category | Vulnerability / Topic | Status |
|-----|-----------|----------|--------------|--------|
| Day 00 | [The Brochure](./Day-00-Brochure/) | OSINT | Information Gathering | ✅ |
| Day 01 | [The Concierge Knows Too Much](./Day-01-Concierge/) | AI | Prompt Injection | ✅ |
| Day 02 | [Room 404](./Day-02-Room404/) | Web | Custom Directory / IDOR | ✅ |
| Day 03 | [Complimentary](./Day-03-Complimentary/) | Cloud | AWS IAM Misconfiguration | ✅ |
| Day 04 | [Packed Light](./Day-04-PackedLight/) | Forensics | PCAP Packet Analysis | ✅ |
| Day 05 | [Beach Bar](./Day-05-BeachBar/) | Boot2Root | Unsafe YAML Deserialization | ✅ |
| Day 06 | [Overheard at Breakfast](./Day-06-OverheardAtBreakfast/) | OSINT | Email Hashing & Gravatar Pivoting | ✅ |
| Day 07 | [Do Not Disturb](./Day-07-DoNotDisturb/) | Boot2Root | NoSQL Bypass & Template Injection | ✅ |
| Day 08 | [Towel on the Sunbed](./Day-08-TowelOnSunbed/) | Web | Race Condition | ✅ |
| Day 09 | [CryptoCabana](./Day-09-CryptoCabana/) | Cloud | Client-Side Secret Exposure | ✅ |
| Day 10 | [The Hollow Shell](./Day-10-HollowShell/) | Web | Zip Slip + RCE | ✅ |
| Day 11 | Infinity Pool | — | — | 🔒 |
| Day 12 | — | — | — | 🔒 |
| Day 13 | — | — | — | 🔒 |
| Day 14 | — | — | — | 🔒 |

> ✅ Completed · 🔒 Not yet released

---

## 📂 Repo Structure
THM-HackerHolidays-2026/
│
├── README.md                          ← You are here
│
├── Day-00-Brochure/
├── Day-01-Concierge/
├── Day-02-Room404/
├── Day-03-Complimentary/
├── Day-04-PackedLight/
├── Day-05-BeachBar/
├── Day-06-OverheardAtBreakfast/
├── Day-07-DoNotDisturb/
├── Day-08-TowelOnSunbed/
├── Day-09-CryptoCabana/
└── Day-10-HollowShell/
## 🧠 Skills Learned

| Skill | Room |
|-------|------|
| OSINT & Info Gathering | Day 00 — The Brochure |
| AI Prompt Injection bypasses | Day 01 — The Concierge Knows Too Much |
| IDOR / Source Web Enumeration | Day 02 — Room 404 |
| AWS IAM Misconfiguration analysis | Day 03 — Complimentary |
| Wireshark PCAP Packet Extraction | Day 04 — Packed Light |
| PyYAML Exploit (Reverse Shell) | Day 05 — Beach Bar |
| Metadata & Gravatar OSINT hunting | Day 06 — Overheard at Breakfast |
| NoSQL Injection & Disk-Group PrivEsc | Day 07 — Do Not Disturb |
| Race Condition exploitation | Day 08 — Towel on the Sunbed |
| Azure Storage & Key Vault version recovery | Day 09 — CryptoCabana |
| Zip Slip path traversal & RCE hooks | Day 10 — The Hollow Shell |
---

## 🔑 Key Takeaways

> These are the most important security lessons from completing these rooms:

1. **Never store secrets in client-side code** — JavaScript is public
2. **Race conditions exist when check and update aren't atomic** — always use transactions
3. **Sanitize ZIP file paths before extraction** — Zip Slip is real and dangerous
4. **Auto-executing uploaded files is extremely dangerous** — sandbox everything
5. **Rotate secrets properly** — old versions in Key Vault stay readable
6. **Least privilege matters** — overprivileged tokens open entire systems
7. **HTML comments are public** — never put credentials in comments

---
## 🕵️ Hidden Lore

While playing through the event, some hidden clues were discovered:

### 🐚 Base64 in the Shell (Main Page)
A Base64 string hidden inside the decorative shell image on the main page decoded to:
> *"JD was never a bug. wt was the business model."*

### 📍 Hidden Coordinates
Found on the main page: `9.5681° N, 100.0602° E`
Along with the message: `> DIG DEEPER.`

These coordinates point to **Ko Samui, Thailand** — the likely fictional location of the Byte Lotus Hotel. 🏝️

### 👤 The Mystery of JD
- VERA's guest list includes: Ponzi, Vibe, Patch, Lambo (@0xMia)
- **JD appears nowhere** in VERA's system
- - The lore suggests JD is the hidden mastermind behind the hotel's criminal operation

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Port scanning |
| Wireshark | PCAP Analysis |
| Firefox DevTools | Source inspection |
| Azure Cloud Shell | Azure CLI commands |
| Python 3 | Payload crafting |
| Netcat | Reverse shell listener |
| Burp Suite | Web traffic analysis |
| dig | DNS enumeration |
| base64 | Decoding hidden messages |

---
## 📜 Writeups

Each room folder contains a detailed README with:
- Full attack chain explanation
- All commands used
- Screenshots at each step
- Vulnerability explanations
- Remediation advice

---

## ⚠️ Disclaimer

These writeups are for **educational purposes only**. All challenges were completed in the official TryHackMe lab environment. Never use these techniques on systems you don't own or have explicit permission to test.

---

## 🔗 Links

- [TryHackMe Hacker Holidays](https://tryhackme.com)

---

*Made with ☕ and curiosity | Updated daily during the event*

