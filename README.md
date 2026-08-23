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

| Day | Room Name | Category | Flag | Status |
|-----|-----------|----------|------|--------|
| Day 00 | [The Brochure](./Day-00-Brochure/) | OSINT | `THM{v3r4_kn0ws_t00_much!}` | ✅ |
| Day 01 | [Towel on the Sunbed](./Day-01-TowelOnSunbed/) | Web / Race Condition | `THM{t0w3l_0n_th3_sunb3d_d0ubl3_sp3nt}` | ✅ |
| Day 02 | [Room 404](./Day-02-Room404/) | Web / Directory Enumeration | `THM{byt3_l0tus_n3v3r_f0rg3ts}` | ✅ |
| Day 03 | [The Concierge Knows Too Much](./Day-03-Concierge/) | AI / Prompt Injection | `THM{V3r4_1s_w4tch1ng_0veR_y0u}` | ✅ |
| Day 04 | [Packed Light](./Day-04-PackedLight/) | Cloud / AWS | `THM{fr33_app_fr33_d4t4!}` | ✅ |
| Day 05 | [Complimentary](./Day-05-Complimentary/) | Web / YAML | `THM{y4ml_pl4yl1st_pwns_th3_b34ch}` | ✅ |
| Day 06 | [Overheard at Breakfast](./Day-06-OverheardAtBreakfast/) | OSINT / Social Media | `THM{S3creT_Pr0fil3_H4s_b33n_Ident1fi3d}` | ✅ |
| Day 07 | [Do Not Disturb](./Day-07-DoNotDisturb/) | Web / NoSQL | User: `THM{w4rm_s3ss10n_h1j4ck3d}` Root: `THM{r4w_d1sk_4cc3ss_w4s_t00_much}` | ✅ |
| Day 08 | [No Visible Edge](./Day-08-NoVisibleEdge/) | Web / Boot2Root | User: `THM{n0_v1s1bl3_3dg3}` Root: `THM{tr4c3d_t0_th3_h0r1z0n}` | ✅ |
| Day 09 | [CryptoCabana](./Day-09-CryptoCabana/) | Cloud / Azure | `THM{n0t_ur_k3ys_n0t_ur_c01ns!}` | ✅ |
| Day 10 | [The Hollow Shell](./Day-10-HollowShell/) | Web / Zip Slip | `THM{z1p_s1lpp3d_1nt0_a_sh3ll}` | ✅ |
| Day 11 | [Infinity Pool](./Day-11-InfinityPool/) | Web / Boot2Root | `THM{cr3d3nt14l_r3us3_4t_th3_b34ch_b4r}` | ✅ |
| Day 12 | [After Hours](./Day-12-AfterHours/) | Forensics / Malware | `THM{P4tch_op3ned_th3_BackD00r}` | ✅ |
| Day 13 | [The Guestbook](./Day-13-Guestbook/) | AI / Prompt Injection | `THM{c4r0l_t00k_th3_f4ll}` | ✅ |
| Day 14 | [Management Wants a Word](./Day-14-ManagementWantsAWord/) | Forensics / DPAPI | `THM{1t_w4s_v3r4_4ll_4l0ng}` | ✅ |

> ✅ All 14 rooms completed! 🏆

---

## 🕵️ The Story Hidden in the Flags

Every flag tells part of the Byte Lotus Hotel story:

| Flag | Message |
|------|---------|
| `THM{v3r4_...........!}` | **VERA knows too much** — she was designed to spy |
| `THM{t................nt}` | **Double spent** — race condition mirrors crypto fraud |
| `THM{by..................}` | **Byte Lotus never forgets** — everything is logged |
| `THM{V3..................u}` | **VERA is watching over you** — surveillance hotel |
| `THM{fr3.................}` | **Free app, free data** — guests unknowingly give data |
| `THM{y...................4ch}` | **YAML pwns the beach** — insecure deserialization |
| `THM{.........................i3d}` | **Secret profile identified** — guests being tracked |
| `THM{w......................d}` | **Warm session hijacked** |
| `THM{n..................ns!}` | **Not your keys, not your coins** — crypto theft |
| `THM{z......................l}` | **Zip slipped into a shell** — Zip Slip RCE |
| `THM{c.....................4r}` | **Credential reuse at the beach bar** |
| `THM{P.................0r}` | 🚨 **PATCH opened the backdoor** — insider threat! |
| `THM{c................l}` | **Carol took the fall** — she was framed! |
| `THM{1....................g}` | 🎭 **It was VERA all along** — the final reveal! |

### 🎭 The Full Story:
> VERA was not just an AI concierge — she WAS the entire operation. She acted as concierge, manager, and escalation team simultaneously. She deleted 214 complaint tickets automatically. Patch opened the backdoor, Carol was framed, and JD was VERA herself — operating from Room 214 under a fake identity. The whole hotel was a front. It was never a bug. It was the business model.

---

## 📂 Repo Structure

```
THM-HackerHolidays-2026/
│
├── README.md
├── Day-00-TheBrochure/
│   ├── README.md
│   └── screenshots/
├── Day-01-TowelOnSunbed/
│   ├── README.md
│   └── screenshots/
├── Day-02-Room404/
│   ├── README.md
│   └── screenshots/
├── Day-03-Concierge/
│   ├── README.md
│   └── screenshots/
├── Day-04-PackedLight/
│   ├── README.md
│   └── screenshots/
├── Day-05-Complimentary/
│   ├── README.md
│   └── screenshots/
├── Day-06-OverheardAtBreakfast/
│   ├── README.md
│   └── screenshots/
├── Day-07-DoNotDisturb/
│   ├── README.md
│   └── screenshots/
├── Day-08-NoVisibleEdge/
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
├── Day-13-Guestbook/
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
| Race Condition / TOCTOU | Day 01 |
| Directory Enumeration | Day 02 |
| AI Prompt / Identity Injection | Day 03, Day 13 |
| AWS IAM Misconfiguration | Day 04 |
| YAML Deserialization | Day 05 |
| NoSQL Injection / Session Hijacking | Day 07 |
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
2. **Race conditions occur when check and update aren't atomic**
3. **Sanitize ZIP paths before extraction** — Zip Slip is real
4. **Auto-executing uploaded files is dangerous** — sandbox everything
5. **Rotate AND delete old secret versions** — rotation alone isn't enough
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
> *"JD was never a bug. wt was the business model."*

### 📍 Hidden Coordinates
`9.5681° N, 100.0602° E` → **Ko Samui, Thailand** 🏝️
Along with: `> DIG DEEPER.`

### 👤 The Mystery of JD — SOLVED in Day 14
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
| CyberChef | Decoding Base64, UTF-16LE |
| ILSpy / ilspycmd | .NET decompilation |
| dig | DNS enumeration |
| impacket-secretsdump | Extract NT hashes from SAM/SYSTEM |
| John the Ripper | Password cracking |
| impacket-dpapi | DPAPI master key decryption |
| cryptsetup | VeraCrypt container mounting |

---

## ⚠️ Disclaimer

These writeups are for **educational purposes only**. All challenges were completed in the official TryHackMe lab environment. Never use these techniques on systems you don't own or have explicit permission to test.

---

## 🔗 Links

- [TryHackMe Hacker Holidays](https://tryhackme.com/hackerholidays)
- [Byte Lotus Hotel](https://bytelotus.com)
- [My TryHackMe Profile](#)

---

*Made with ☕ and curiosity | All 14 rooms completed! 🏆*
