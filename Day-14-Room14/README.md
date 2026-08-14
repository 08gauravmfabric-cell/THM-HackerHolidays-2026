# 🏢 Management Wants a Word — TryHackMe Writeup
**Hacker Holidays 2026 | Day 14 | Byte Lotus Hotel**

![Difficulty](https://shields.io)
![Category](https://shields.io)
![Status](https://shields.io)

---

## 📋 Room Info

| Field | Details |
|-------|---------|
| **Room Name** | Management Wants a Word |
| **Event** | Hacker Holidays 2026 |
| **Day** | 14 of 14 |
| **Category** | Windows Forensics / Credential Recovery / Decryption |
| **Vulnerabilities** | Artifact Extraction · DPAPI Credential Exposure · Registry Hive Leak · Volume Misconfiguration |
| **Tools Used** | Impacket (secretsdump) · Mimikatz · Python · VeraCrypt · John the Ripper / Hashcat |
| **Flags** | Final Room Flag |

---

## 🧠 What We Learned

- Offline Windows hive extraction allows parsing of full NTLM password hashes from memory templates.
- DPAPI (Data Protection API) relies on domain/local master keys that can be completely reversed with boot keys.
- Web browsers like Chrome store passwords locally using DPAPI structures which are vulnerable if system hives are acquired.
- Encrypted virtual disks (like VeraCrypt volumes) require strong passphrase generation to withstand offline dictionary sweeps.
- Final incident milestones require cross-referencing multiple forensic fragments to rebuild administrative targets.

---
## 🗺️ Attack Chain Overview
---

## 🔍 Step 1 — Local Artifact Assembly

We begin our investigation by downloading the forensic acquisition package provided for Day 14. Extracting the file structures yields standard offline Windows OS artifacts:

- Windows Registry Hives: `SAM`, `SYSTEM`, `SECURITY`
- Application Data Path: `\AppData\Local\Google\Chrome\User Data\`
- Encrypted Virtual Container: `secure_storage.vc`

```bash
unzip management_secrets.zip -d forensic_analysis/
cd forensic_analysis/
```

> 📸 **Screenshot:** `screenshots/01_artifact_extraction.jpg`

---

## 🔍 Step 2 — Registry Hash Dumping

With direct offline access to the core registry database components, we run `secretsdump.py` from the Impacket suite to parse the system boot key and extract local account hashes.

```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
```

The tool successfully extracts the operational administrator NTLM strings:
- **Administrator:** `500:AAD3B435B51404EEAAD3B435B51404EE:31D6CFE0D16AE931B73C59D7E0C089C0:::`
- **Manager Account:** `1002:AAD3B435B51404EEAAD3B435B51404EE:D14B892A829D1047A221C18D16F1E3A2:::`

> 📸 **Screenshot:** `screenshots/02_secretsdump_output.jpg`

---

## 🔍 Step 3 — Cracking the Manager's NTLM Hash

To pivot deeper into user space artifacts, we feed the Manager's NTLM string into Hashcat using the standard rockyou wordlist dictionary to extract the raw text login key.

```bash
hashcat -m 1000 d14b892a829d1047a221c18d16f1e3a2 /usr/share/wordlists/rockyou.txt
```

The cracking core maps the hash back to the cleartext string value:
- **Manager Password:** `bylotus_mgnt_2026!`

> 📸 **Screenshot:** `screenshots/03_hashcat_cracked.jpg`

---

## 🔍 Step 4 — Extracting DPAPI Master Keys

Chrome protects credentials stored inside its internal database using the Windows Data Protection API (DPAPI). To decrypt them offline, we extract the local master key using the cracked password and the account's internal SID parameters.

Using forensic scripts or launching a temporary emulation layer, we pass the user's master key master file located inside the profile's system directory:

```text
Path: \AppData\Roaming\Microsoft\Protect\<USER_SID>\
```

Using Mimikatz/Python-DPAPI bindings:
```bash
dpapi::masterkey /in:"forensic_analysis/Protect/MGR_SID_PATH" /sid:S-1-5-21-... /password:bylotus_mgnt_2026!
```

This returns the verified hex master key context required to unlock local credential strings.

> 📸 **Screenshot:** `screenshots/04_dpapi_masterkey.jpg`

---

## 🔍 Step 5 — Decrypting the Chrome Login Database

We locate the target credential file at its native structural application location:
```text
\AppData\Local\Google\Chrome\User Data\Default\Login Data
```

This file is a standard SQLite database wrapper. We execute an extraction script using our recovered master key to pull and parse the target application logins:

```bash
python3 decrypt_chrome.py -db "Login Data" -key <DECRYPTED_DPAPI_HEX_KEY>
```

The database output prints accounts tied to internal platforms, exposing the administration console login properties:

```json
{
  "url": "https://bytelotus.thm",
  "username": "hotel_director",
  "password_decrypted": "VeraCrypt_Master_Pass_99#"
}
```

> 📸 **Screenshot:** `screenshots/05_chrome_decrypted.jpg`

---

## 🔍 Step 6 — Mounting the VeraCrypt Container

The final project objective is an isolated binary storage vault asset labeled `secure_storage.vc`. Using the credential string recovered from the local browser extraction loop, we attempt an administrative storage mount.

1. Launch your local console environment or open the GUI suite for **VeraCrypt**.
2. Select **Select File** and target the absolute path of `secure_storage.vc`.
3. Select an unassigned drive slot index from the grid display panel.
4. Click **Mount**.
5. Input the extracted password string when prompted: `VeraCrypt_Master_Pass_99#`.

The operation finishes successfully, attaching an automated cleartext file system sector onto the workstation.

> 📸 **Screenshot:** `screenshots/06_veracrypt_mount.jpg`

---

## 🔍 Step 7 — Final Flag Recovery

Navigate into the root directory path of the newly exposed virtual storage partition allocation:

```bash
cd /media/veracrypt1/
ls -la
cat flag.txt
```

🎉 **The final event validation flag appears cleanly inside the text output stream!**

To maintain absolute compliance with TryHackMe walkthrough formatting policies, the literal flag string value is masked below:

* **Final Flag:** `THM{REDACTED_MANAGEMENT_ULTIMATE_COMPROMISE_SUCCESS}`

> 📸 **Screenshot:** `screenshots/07_final_flag.jpg`

---

## 🛡️ Vulnerability Summary

| # | Vulnerability | Location | Impact | Remediation |
|---|--------------|----------|--------|-------------|
| 1 | Insecure local registry asset retention | System Backups / Memory dumps | System-wide local hash disclosure | Enforce strict file system permissions on configuration checkpoints and clear old raw backup states. |
| 2 | Low-entropy local password selection | Manager Windows Account | Rapid offline dictionary discovery | Implement active enterprise Group Policy rules requiring strong, random administrative characters. |
| 3 | Cleartext session browser credential sync | Chrome Data Profile | Local system password exposure | Use hardware-backed security modules (like TPM) to isolate local key storage away from standard file paths. |
| 4 | Static password asset recycling | Storage Containers | Multi-layer compromise paths | Implement dynamic key tracking rules and ensure passwords are never shared between internal networks and localized disk containers. |



