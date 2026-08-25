# 🏖️ TryHackMe — Hacker Holidays 2026
> *"Everyone's on holiday. Everyone's hiding something."*

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red)
![Event](https://img.shields.io/badge/Event-Hacker%20Holidays%202026-orange)
![Rooms](https://img.shields.io/badge/Rooms-14%20Days-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 🏨 About the Event

**Hacker Holidays 2026** is a free 14-day cybersecurity challenge hosted by TryHackMe, set inside the fictional **Byte Lotus Hotel** — a five-star resort with a zero-star security posture.

Starting **July 27, 2026**, a new room unlocked daily at **4PM UTC**, covering:

- 🔍 OSINT · 🌐 Web Exploitation · ☁️ Cloud Security · 🔬 Digital Forensics · 🤖 AI Security

Every completed room earned a raffle ticket for the **$50,000+ prize pool**.

---

## 🗺️ Room Progress

| Day | Room Name | Category | Vulnerability | Status |
|-----|-----------|----------|--------------|--------|
| Day 00 | [The Brochure](./Day-00-Brochure/) | OSINT | Information Gathering | ✅ |
| Day 01 | [The Concierge Knows Too Much](./Day-01-Concierge/) | AI | Prompt Injection / Identity Spoofing | ✅ |
| Day 02 | [Room 404](./Day-02-Room404/) | Web | Directory Enumeration | ✅ |
| Day 03 | [Complimentary](./Day-03-Complimentary/) | Cloud | AWS IAM Misconfiguration | ✅ |
| Day 04 | [Packed Light](./Day-04-PackedLight/) | Forensics | Network Traffic Analysis | ✅ |
| Day 05 | [Beach Bar](./Day-05-BeachBar/) | Web | YAML Deserialization / RCE | ✅ |
| Day 06 | [Overheard at Breakfast](./Day-06-OverheardAtBreakfast/) | OSINT | Social Media / Email OSINT | ✅ |
| Day 07 | [Do Not Disturb](./Day-07-DoNotDisturb/) | Web | NoSQL Injection / Session Hijacking | ✅ |
| Day 08 | [Towel on the Sunbed](./Day-08-TowelOnSunbed/) | Web | Race Condition / TOCTOU | ✅ |
| Day 09 | [CryptoCabana](./Day-09-CryptoCabana/) | Cloud | Client-Side Secret Exposure / Azure | ✅ |
| Day 10 | [The Hollow Shell](./Day-10-HollowShell/) | Web | Zip Slip / RCE | ✅ |
| Day 11 | [Infinity Pool](./Day-11-InfinityPool/) | Web | Boot2Root / Credential Reuse | ✅ |
| Day 12 | [After Hours](./Day-12-Room12/) | Forensics | WMI Persistence / Fileless Malware | ✅ |
| Day 13 | [The Guestbook](./Day-13-Room13/) | AI | Prompt Injection / Command Injection | ✅ |
| Day 14 | [Management Wants a Word](./Day-14-Room14/) | Forensics | DPAPI / Chrome Passwords / VeraCrypt | ✅ |

> ✅ All 14 rooms completed! 🏆

---

## 🕵️ The Story Hidden in the Flags

Each flag contains a hidden message that tells the full Byte Lotus Hotel story:

| Day | Room | Hidden Message |
|-----|------|---------------|
| Day 00 | The Brochure | VERA knows too much |
| Day 01 | The Concierge Knows Too Much | VERA is watching over you |
| Day 02 | Room 404 | Byte Lotus never forgets |
| Day 03 | Complimentary | Free app, free data |
| Day 04 | Packed Light | Traced to the horizon |
| Day 05 | Beach Bar | YAML playlist pwns the beach |
| Day 06 | Overheard at Breakfast | Secret profile has been identified |
| Day 07 | Do Not Disturb | Warm session hijacked |
| Day 08 | Towel on the Sunbed | Towel on the sunbed — double spent |
| Day 09 | CryptoCabana | Not your keys, not your coins |
| Day 10 | The Hollow Shell | Zip slipped into a shell |
| Day 11 | Infinity Pool | Credential reuse at the beach bar |
| Day 12 | After Hours | 🚨 Patch opened the backdoor |
| Day 13 | The Guestbook | Carol took the fall |
| Day 14 | Management Wants a Word | 🎭 It was VERA all along |

### 🎭 The Full Story:
> VERA was not just an AI concierge — she WAS the entire operation. She acted as concierge, manager, and escalation team simultaneously. She deleted 214 complaint tickets automatically. Patch opened the backdoor, Carol was framed, and JD was VERA herself — operating from Room 214 under a fake identity. The whole hotel was a front. It was never a bug. It was the business model.

---

## 📂 Repo Structure

```
THM-HackerHolidays-2026/
│
├── README.md
├── Day-00-Brochure/
│   ├── README.md
│   └── screenshots/
├── Day-01-Concierge/
│   ├── README.md
│   └── screenshots/
├── Day-02-Room404/
│   ├── README.md
│   └── screenshots/
├── Day-03-Complimentary/
│   ├── README.md
│   └── screenshots/
├── Day-04-PackedLight/
│   ├── README.md
│   └── screenshots/
├── Day-05-BeachBar/
│   ├── README.md
│   └── screenshots/
├── Day-06-OverheardAtBreakfast/
│   ├── README.md
│   └── screenshots/
├── Day-07-DoNotDisturb/
│   ├── README.md
│   └── screenshots/
├── Day-08-TowelOnSunbed/
│   ├── README.md
│   └── screenshots/
├── Day-09-CryptoCabana/
│   ├── README.md
│   └── screenshots/
├── Day-10-HollowShell/
│   ├── README.md
│   └── screenshots/
├── Day-11-InfinityPool/
│   ├── README.md
│   └── screenshots/
├── Day-12-AfterHours/
│   ├── README.md
│   └── screenshots/
├── Day-13-TheGuestbook/
│   ├── README.md
│   └── screenshots/
└── Day-14-ManagementWantsAWord/
    ├── README.md
    └── screenshots/
```

---

## 🧠 Skills Learned

| Skill | Room |
|-------|------|
| OSINT & Information Gathering | Day 00, Day 06 |
| AI Prompt Injection / Identity Spoofing | Day 01, Day 13 |
| Directory Enumeration | Day 02 |
| AWS IAM Misconfiguration | Day 03 |
| Network Traffic / PCAP Analysis | Day 04 |
| YAML Deserialization / RCE | Day 05 |
| NoSQL Injection / Session Hijacking | Day 07 |
| Race Condition / TOCTOU | Day 08 |
| Azure Storage Enumeration | Day 09 |
| Client-Side Secret Exposure | Day 09 |
| Azure Key Vault Version Recovery | Day 09 |
| Zip Slip Path Traversal | Day 10 |
| Remote Code Execution | Day 10, Day 11 |
| SSH Tunneling | Day 11 |
| FreePBX UCP Exploitation | Day 11 |
| WMI Forensics / Malware Analysis | Day 12 |
| .NET Reverse Engineering | Day 12 |
| Command Injection via AI Agent | Day 13 |
| Windows KAPE Forensics | Day 14 |
| DPAPI Master Key Decryption | Day 14 |
| Chrome Password Extraction | Day 14 |
| VeraCrypt Container Analysis | Day 14 |

---

## 🔑 Key Security Takeaways

1. **Never store secrets in client-side code** — JS is public
2. **Race conditions occur when check and update are not atomic**
3. **Sanitize ZIP paths before extraction** — Zip Slip is real
4. **Auto-executing uploaded files is dangerous** — sandbox everything
5. **Rotate AND delete old secret versions** — rotation alone is not enough
6. **Least privilege matters** — overprivileged tokens open entire systems
7. **HTML comments are public** — never put credentials there
8. **WMI is a common malware hiding spot** — include it in forensics
9. **AI agents must not treat user input as instructions**
10. **Authorization must be in code, not natural language**
11. **Output redaction is not a security control** — encoding bypasses it
12. **Never store secrets in system prompts** — AI can be tricked to reveal them
13. **DPAPI protects credentials but keys can be derived from password**
14. **Browser saved passwords are a goldmine for forensic investigators**

---

## 🕵️ Hidden Lore Discoveries

### 🐚 Base64 in the Shell
Hidden inside the decorative shell image on the main page decoded to:
> *"JD was never a bug. It was the business model."*

### 📍 Hidden Coordinates
`9.5681° N, 100.0602° E` → **Ko Samui, Thailand** 🏝️
Along with: `> DIG DEEPER.`

### 👤 The Mystery of JD — Solved in Day 14
JD was VERA all along — operating from Room 214 under a fake identity. The laptop left behind in Room 214 was VERA's own machine containing the financial documents that proved the hotel was a criminal front.

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Port scanning |
| Firefox DevTools | Source inspection |
| Azure Cloud Shell | Azure CLI commands |
| Python 3 | Payload crafting / decompression |
| Netcat | Reverse shell listener |
| SSH | Tunneling to internal services |
| strings / grep | Binary analysis |
| CyberChef | Decoding Base64 / UTF-16LE |
| ILSpy / ilspycmd | .NET decompilation |
| dig | DNS enumeration |
| impacket-secretsdump | Extract NT hashes from SAM/SYSTEM |
| John the Ripper | Password cracking |
| impacket-dpapi | DPAPI master key decryption |
| cryptsetup | VeraCrypt container mounting |
| Wireshark | PCAP analysis |

---

## ⚠️ Disclaimer

These writeups are for **educational purposes only**. All challenges were completed in the official TryHackMe lab environment. Never use these techniques on systems you do not own or have explicit permission to test.

> 🚫 Flags are intentionally not shown in compliance with TryHackMe's writeup policy.

---

## 🔗 Links

- [TryHackMe Hacker Holidays](https://tryhackme.com/hackerholidays)



---

*Made with ☕ and curiosity | All 14 rooms completed! 🏆* And after it got the completion certificate.
