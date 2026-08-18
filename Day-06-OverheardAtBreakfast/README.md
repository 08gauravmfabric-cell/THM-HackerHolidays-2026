# 🍳 Overhead Breakfast — TryHackMe Hacker Holidays 2026

> **Day 6 of the Hacker Holidays 2026 event**
> Category: OSINT | Difficulty: Easy

---

## 📋 Room Summary

| Field | Details |
|---|---|
| **Room Name** | Overhead Breakfast |
| **Event** | TryHackMe Hacker Holidays 2026 |
| **Difficulty** | Easy |
| **Type** | OSINT / Social Media Investigation |
| **Skills** | Email OSINT, Gravatar, Base64 Decoding |
| **Flag** | ✅ Captured |

---

## 🗺️ Attack Chain Overview

```
Task Files Downloaded
    └─► Conversation Screenshot Analysis
            └─► Email Address Extracted
                    └─► lambobytelotushotel@gmail.com
                            └─► Epieos Reverse Email Lookup
                                    └─► Google Account Found
                                            └─► Gravatar Profile Discovered
                                                    └─► Base64 Encoded Prize String
                                                            └─► CyberChef Decode
                                                                    └─► Flag ✅
```

---

## 🔍 Step-by-Step Walkthrough

### Step 1: Download and Analyze Task Files

The room provides a downloadable task file containing a **screenshot of a conversation** between two guests at the Byte Lotus Hotel — Ponzi (an influencer) and Lambo.

**Key details from the conversation:**
- Lambo mentions staying at **Byte Lotus Hotel**
- Lambo mentions using a free tool "starting with G" to link social media accounts
- Lambo provides their email: **`lambobytelotushotel@gmail.com`**
- Lambo mentions they "wiped everything" from their social media

> 💡 **Clue:** "Started with a G if I remember correctly" → **Gravatar** (a profile service starting with G that links email to social accounts)

---

### Step 2: Email OSINT with Epieos

Using the email address found in the conversation, we performed a **reverse email lookup** using Epieos:

```
Target Email: lambobytelotushotel@gmail.com
Tool: https://epieos.com
```

**Results from Epieos:**

| Service | Finding |
|---|---|
| Google Account ID | `109541557676124188877` |
| Google Maps | Profile with contributions |
| Google Calendar | Public calendar link |
| Google Plus Archive | Wayback Machine archive |

---

### Step 3: Gravatar Profile Discovery

Since Lambo mentioned a tool "starting with G" that links accounts via email/profile, we searched **Gravatar** — a service that creates a global profile linked to an email address:

```
https://gravatar.com/lambobytelotushotel
```

or search by email hash on Gravatar's lookup:

```
https://en.gravatar.com/emails/
```

**Gravatar Profile Found:**
- **Name:** Lambo
- **Bio:** Lam-boh · Byte Lotus Hotel
- **Prize:** A Base64 encoded string hidden in the profile bio!

```
VEhNe1MzY3JIVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9
```

---

### Step 4: Decode the Flag with CyberChef

The prize string is **Base64 encoded**. Decode it using CyberChef or the command line:

**Using CyberChef:**
1. Go to `https://gchq.github.io/CyberChef/`
2. Paste the string in the Input box
3. Search for **"From Base64"** recipe
4. Apply and read the output

**Using command line:**
```bash
echo "VEhNe1MzY3JIVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9" | base64 -d
```

**Result:**
```
THM{S3cr3T_Pr0fil3_H4s_b33n_Id3nt1fi3d}
```

---

## 🧠 OSINT Techniques Used

| Technique | Tool | Purpose |
|---|---|---|
| Reverse Email Lookup | Epieos | Find linked accounts from email |
| Google Account Enumeration | Epieos | Find Google ID and services |
| Gravatar Profile Search | gravatar.com | Find linked profile with hidden data |
| Base64 Decoding | CyberChef / bash | Decode the prize string |

---

## 💡 Key OSINT Principles Demonstrated

### 1. Email as an Identity Anchor
An email address is one of the most powerful OSINT pivot points. A single email can link to:
- Google account and all its services
- Gravatar profile
- Social media accounts
- Data breach records

### 2. "Deleted" Doesn't Mean Gone
Lambo said they "wiped everything" — but:
- Gravatar profiles persist even after social media deletion
- Google accounts retain Maps contributions, calendar links
- Wayback Machine archives content from Google Plus

### 3. Profile Services as Hidden Data Stores
Services like Gravatar are often overlooked in OSINT investigations but can contain:
- Real names and usernames
- Linked social accounts
- Bio information and sometimes hidden messages

---

## 🛡️ Lessons Learned

1. **Email addresses are identity anchors** — never share your real email publicly if you want anonymity.
2. **"Deleting" social media doesn't remove all traces** — profile services, archives, and linked accounts persist.
3. **Gravatar links your email to a public profile** — this is searchable and indexable by default.
4. **Reverse email lookup tools** (Epieos, Holehe) can reveal dozens of linked accounts from a single email.
5. **Always check the Google Account ID** — it links to Maps, Calendar, and archived Google Plus profiles.

---

## 🔧 Tools Used

| Tool | Purpose | Link |
|---|---|---|
| Epieos | Reverse email lookup | https://epieos.com |
| Gravatar | Profile lookup by email | https://gravatar.com |
| CyberChef | Base64 decoding | https://gchq.github.io/CyberChef |
| Google Maps | Check contributions | https://maps.google.com |
| Wayback Machine | Archive search | https://web.archive.org |

---

## 📸 Screenshots

> *(Add your screenshots here)*

| Screenshot | Description |
|---|---|
| `conversation.png` | Task file — conversation between Ponzi and Lambo |
| `epieos_result.png` | Epieos reverse lookup results |
| `gravatar_profile.png` | Gravatar profile showing prize string |
| `cyberchef_decode.png` | CyberChef Base64 decode |

---

## 📁 Files

| File | Description |
|---|---|
| `README_OverheadBreakfast.md` | This writeup |
| `epieos_20260801T163154Z_LambobyteLotusHotel_gmail_com.pdf` | Epieos report |

---

## 🔗 References

- [Epieos — Email OSINT Tool](https://epieos.com)
- [Gravatar — Global Avatar Service](https://gravatar.com)
- [CyberChef — The Cyber Swiss Army Knife](https://gchq.github.io/CyberChef)
- [Holehe — Email Account Checker](https://github.com/megadose/holehe)

---

*Writeup by Gaurav | TryHackMe Hacker Holidays 2026*

