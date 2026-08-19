# 💰 Crypto Cabana — TryHackMe Hacker Holidays 2026

> **Hacker Holidays 2026 event**
> Category: Cryptography | Difficulty: Medium

---

## 📋 Room Summary

| Field | Details |
|---|---|
| **Room Name** | Crypto Cabana |
| **Event** | TryHackMe Hacker Holidays 2026 |
| **Difficulty** | Medium |
| **Type** | Cryptography / Web3 Investigation |
| **Target** | CryptoCabana Wallet / Blockchain |
| **Skills** | Blockchain Analysis, Cryptography, Transaction Investigation |
| **Flag** | ✅ Captured |

---

## 🗺️ Attack Chain Overview

```
Room Brief — Unauthorized Wallet Transaction
    └─► Blockchain Transaction Analysis
            └─► Wallet Investigation
                    └─► Transaction: 0x9fa4ce1b
                            └─► Trace Unauthorized Signing
                                    └─► Cryptographic Analysis
                                            └─► Key/Signature Recovery
                                                    └─► Flag ✅
```

---

## 📖 Room Narrative

The room story describes a guest at the Byte Lotus resort whose **CryptoCabana wallet balance changed by -3.4 BTC** via transaction `0x9fa4ce1b` — a transaction they never authorized. The investigation reveals:

- *"A session goes warm on a sunbed, and a stranger sits down in it"* → Session hijacking
- *"A wallet signs a transaction its owner didn't authorize"* → Unauthorized transaction signing
- *"A shell on the beach answers back"* → Remote access

**Key clue from the story:** The wallet owner was at the buffet — someone else signed the transaction using their session or private key.

---

## 🔍 Step-by-Step Walkthrough

### Step 1: Download and Analyze Task Files

The room provides task files containing:
- Wallet transaction details
- Cryptographic challenge data
- Blockchain records

```
Transaction ID: 0x9fa4ce1b
Amount: -3.4 BTC
Wallet: CryptoCabana Wallet
```

---

### Step 2: Transaction Analysis

Investigate the unauthorized transaction:

```bash
# Analyze transaction data
# Document what tools you used here
```

**Key findings:**
- Transaction signed without owner authorization
- Investigate the signing key/method
- Look for session reuse or key exposure

> *(Add your specific findings here)*

---

### Step 3: Cryptographic Investigation

> *(Document your cryptographic analysis steps here)*

**Tools used for crypto analysis:**

```bash
# Add your cryptography commands here
# e.g., openssl, hashcat, john, etc.
```

---

### Step 4: Key Recovery / Exploit

> *(Document how you recovered the key or exploited the vulnerability)*

```bash
# Add your exploitation steps here
```

---

### Step 5: Flag Capture

```
THM{...}
```

Flag captured! 🎉

---

## 🧠 Vulnerability Analysis

### What Went Wrong?

| Issue | Description |
|---|---|
| Session Reuse | An active session was hijacked by another party |
| Unauthorized Signing | Transaction was signed without owner consent |
| *(Add more)* | *(Add description)* |

---

## 💡 Cryptography Concepts Covered

### 1. Digital Signatures
Blockchain transactions are signed using private keys. If a private key or session is compromised, unauthorized transactions can be made on behalf of the owner.

### 2. Transaction Tracing
Blockchain transactions are public and immutable. Every transaction can be traced back to its signing key.

### 3. Session Security
Web3 applications often store private keys or signing sessions in browser storage — these can be stolen via XSS or session hijacking.

---

## 🛡️ Lessons Learned

1. **Never leave active sessions unattended** — session tokens can be stolen and reused.
2. **Hardware wallets** provide the strongest protection against unauthorized signing.
3. **Always verify transaction details** before signing — check recipient, amount, and gas fees.
4. **Private keys should never be stored in browser storage** — use secure keystores.
5. **Multi-signature wallets** require multiple approvals, preventing single-point-of-failure attacks.

---

## 🔧 Tools Used

| Tool | Purpose |
|---|---|
| *(Add tool)* | *(Add purpose)* |
| `openssl` | Cryptographic operations |
| `hashcat` / `john` | Hash cracking if needed |
| Blockchain Explorer | Transaction analysis |
| CyberChef | Encoding/decoding |

---

## 📸 Screenshots

> *(Add your screenshots here)*

| Screenshot | Description |
|---|---|
| `room_overview.png` | Room narrative and wallet transaction |
| `transaction.png` | Unauthorized transaction details (0x9fa4ce1b) |
| `analysis.png` | Cryptographic analysis |
| `flag.png` | Flag captured |

---

## 📁 Files

| File | Description |
|---|---|
| `README_CryptoCabana.md` | This writeup |
| *(task files)* | Downloaded task files |

---

## 🔗 References

- [Blockchain Transaction Analysis](https://www.blockchain.com/explorer)
- [Ethereum Transaction Tracing](https://etherscan.io)
- [CyberChef](https://gchq.github.io/CyberChef)
- [OpenSSL Docs](https://www.openssl.org/docs)
- [Web3 Security Best Practices](https://consensys.github.io/smart-contract-best-practices)

---

*Writeup by Gaurav | TryHackMe Hacker Holidays 2026*
