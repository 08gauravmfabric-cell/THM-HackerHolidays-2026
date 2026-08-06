# 🏖️ TryHackMe — Hacker Holidays 2026
> *"Everyone's on holiday. Everyone's hiding something."*

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red)
![Event](https://img.shields.io/badge/Event-Hacker%20Holidays%202026-orange)
![Rooms](https://img.shields.io/badge/Rooms-14%20Days-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

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

| Day | Room Name | Category | Vulnerability | Status |
|-----|-----------|----------|--------------|--------|
| Day 00 | [The Brochure](#) | OSINT | Information Gathering | ⬜ |
| Day 01 | [Towel on the Sunbed](./Day-01-TowelOnSunbed/) | Web | Race Condition | ✅ |
| Day 02 | [Room 404](./Day-02-Room404/) | Web | — | ✅ |
| Day 03 | [The Concierge Knows Too Much](./Day-03-Concierge/) | AI | Prompt Injection | ⬜ |
| Day 04 | [Complimentary](./Day-04-Complimentary/) | Cloud | AWS IAM Misconfiguration | ⬜ |
| Day 05 | — | — | — | 🔒 |
| Day 06 | — | — | — | 🔒 |
| Day 07 | — | — | — | 🔒 |
| Day 08 | — | — | — | 🔒 |
| Day 09 | [CryptoCabana](./Day-09-CryptoCabana/) | Cloud | Client-Side Secret Exposure | ✅ |
| Day 10 | [The Hollow Shell](./Day-10-HollowShell/) | Web | Zip Slip + RCE | ✅ |
| Day 11 | Infinity Pool | — | — | 🔒 |
| Day 12 | — | — | — | 🔒 |
| Day 13 | — | — | — | 🔒 |
| Day 14 | — | — | — | 🔒 |

> ✅ Completed · ⬜ Not started · 🔒 Not yet released

---

## 📂 Repo Structure

```
THM-HackerHolidays-2026/
│
├── README.md                          ← You are here
│
├── Day-01-TowelOnSunbed/
│   ├── README.md                      ← Room writeup
│   └── screenshots/                   ← Room screenshots
│
├── Day-02-Room404/
│   ├── README.md
│   └── screenshots/
│
├── Day-09-CryptoCabana/
│   ├── README.md
│   └── screenshots/
│
├── Day-10-HollowShell/
│   ├── README.md
│   └── screenshots/
│
└── ...                                ← More rooms added daily
```

---

## 🧠 Skills Learned

| Skill | Room |
|-------|------|
| Race Condition exploitation | Day 01 — Towel on the Sunbed |
| Azure Storage enumeration | Day 09 — CryptoCabana |
| Client-side secret exposure | Day 09 — CryptoCabana |
| Azure Key Vault version recovery | Day 09 — CryptoCabana |
| Hardcoded credential discovery | Day 10 — The Hollow Shell |
| Zip Slip path traversal | Day 10 — The Hollow Shell |
| Remote Code Execution via hooks | Day 10 — The Hollow Shell |
| Reverse shell with Netcat | Day 10 — The Hollow Shell |

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
- The lore suggests JD is the hidden mastermind behind the hotel's criminal operation

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Port scanning |
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

- [TryHackMe Hacker Holidays](https://tryhackme.com/hackerholidays)
- [Byte Lotus Hotel](https://bytelotus.com)
- [My TryHackMe Profile](#)

---

*Made with ☕ and curiosity | Updated daily during the event*
