# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 14 (Management Wants a Word)

## 📖 Introduction & Scenario Context
The grand finale of Hacker Holidays leads to an advanced **Digital Forensics and Incident Response (DFIR)** investigation. Housekeeping at the Byte Lotus Hotel discovered a guest laptop left behind after an early checkout in Room 214, registered under the name "Vera". Before wiping the machine, IT pulled a full KAPE triage capture folder. 

Our objective is to deeply audit the local Windows forensics artifacts, extract hidden security keys via Data Protection API (DPAPI) bypasses, decrypt password storage components, and unpack an encrypted virtual volume to expose the final conspiracy flag.

---

## 🛠️ Step 1: Initial Triage Inspection & Account Extraction
We begin our forensic analysis by auditing the provided KAPE triage file system structure to pull security identifiers and local user hashes.

1. **Extracting the Forensic Triage Data:** Extract the target file archive locally onto your analysis machine:
   ```bash
   tar -xf triage_evidence.tar.gz -C ./evidence/
   ```
2. **Locating Core Hive Registries:** Navigate into the directory containing core Windows configuration hives: `C/Windows/System32/config/`. Locate the **SAM**, **SECURITY**, and **SYSTEM** database tables.
3. **Dumping Local User Secrets:** Use Impacket's `secretsdump.py` tool locally to parse out the Security Account Manager (SAM) databases to acquire the NTLM hash of the local account profile `Vera`:
   ```bash
   python3 secretsdump.py -sam ./evidence/C/Windows/System32/config/SAM -system ./evidence/C/Windows/System32/config/SYSTEM local
   ```
4. **Recording User Credentials:** Note the resulting NTLM hash string printed for user Vera. This hash is vital for reversing Windows internal DPAPI storage encryption protections.
   - Target Identified: `Vera:1001:aad3b435b51404eeaad3b435b51404ee:b2c89f...` *(Example structural hash)*

---

## 🔍 Step 2: Decrypting the DPAPI Master Key
Google Chrome and Windows internal mechanisms safeguard locally saved passwords utilizing a security pipeline protected by the individual user's DPAPI configurations.

1. **Locating User MasterKeys:** Navigate directly to the target user's roaming application credential storage paths:
   ```bash
   cd ./evidence/C/Users/Vera/AppData/Roaming/Microsoft/Protect/
   ```
2. **Identifying the Target GUID Folder:** Look inside the directory to find a long security identifier naming string containing the active raw binary master key block file.
3. **Decrypting the MasterKey Block:** Run `dpapick` or specialized Mimikatz modules to parse the binary blob. Using the extracted NTLM master hash string from Step 1, execute the conversion routine to unpack the master key parameter:
   ```bash
   python3 dpapick.py --mk ./Microsoft/Protect/<GUID>/<KEY_FILE> --ntlm b2c89f...
   ```
4. **Acquiring the Plaintext MasterKey:** Save the resulting clean hex-encoded plaintext master key string. This master token will now unlock application-specific vaults on this profile.

---

## 🚀 Step 3: Extracting Chrome Credentials & Login Secrets
With the underlying DPAPI structure defeated, we extract Chrome's internal configuration state profiles to access saved website credentials.

1. **Locating the Chrome State File:** Navigate to the browser's profile tracking home:
   ```bash
   cd ./evidence/C/Users/Vera/AppData/Local/Google/Chrome/User Data/
   ```
2. **Extracting the Local State Key:** Open the file named `Local State` and extract the base64-encoded string assigned to the parameter `os_crypt.encrypted_key`. 
3. **Unwrapping the Chrome Encryption Token:** Pass the encrypted key block together with your decrypted DPAPI MasterKey from Step 2 into your forensic software framework tool to yield the absolute plaintext symmetric AES-256 key utilized by Google Chrome.
4. **Querying the SQLite Login Database:** Chrome saves website passwords inside an SQLite database folder located at `Default/Login Data`. Open the database using an internal terminal reader:
   ```bash
   sqlite3 "./Default/Login Data" "SELECT origin_url, username_value, password_value FROM logins;"
   ```
5. **Decrypting the Password Column:** Take the raw blob data extracted from the `password_value` data field and run the symmetric decryption pipeline using the recovered Chrome AES key. This exposes an administrative storage password string.
   - Decrypted Target Master String: `B3achL0tusP@ss2026!` *(Example extracted value)*

---

## 🔓 Step 4: Mounting and Unlocking the VeraCrypt Volume
The triage file structure reveals a hidden backup container asset that requires third-party mounting.

1. **Locating the Encrypted Target Volume:** Search the user's local documents or desktop folders to isolate a file named `backup.vc` or a structural equivalent.
2. **Setting up the Decryption Engine:** Launch your local console **VeraCrypt** module interface or call the command line utility wrapper.
3. **Mounting the File Asset:** Map the file out to an unallocated local media slot path using the decrypted credential key string from Step 3:
   ```bash
   veracrypt --text --non-interactive --mount ./evidence/C/Users/Vera/Documents/backup.vc /mnt/forensic_vault/ --password "B3achL0tusP@ss2026!"
   ```
4. **Auditing Content Files:** Change directories directly into the securely opened loop mount directory interface area:
   ```bash
   cd /mnt/forensic_vault/
   ls -la
   ```
5. **Inspecting the Forensic PDF:** Locate a confidential document file asset (e.g., `hotel_conspiracy_manifest.pdf`).

---

## 🧩 Step 5: Advanced OCR Text Extraction Chain
The document appears to protect its raw text fields by rendering data inside flat high-resolution image matrices to block simple string search tools.

1. **Converting Document Layers into Text:** Use the `tesseract` Optical Character Recognition (OCR) engine script tool to capture character configurations cleanly:
   ```bash
   tesseract hotel_conspiracy_manifest.pdf stdout --dpi 300 | grep "THM{"
   ```
2. **Final Validation:** The system reads the image canvas layers, directly resolving the embedded text blocks into the structured flag target string.

---

## 🏁 Flag Capture
To explicitly maintain absolute compliance with TryHackMe walkthrough platform deployment frameworks and solution protection policies, the literal flag text string is securely hidden below:

* **Flag:** `THM{REDACTED_FINALE_DPAPI_CHROME_VERACRYPT_FORENSICS_MASTERED}`

---

## 🛡️ Strategic Mitigation Actions
* **Enforce Enterprise Credential Management:** Local browser credential storage spaces are susceptible to offline machine triage dumps. Organizations must transition user profiles away from standard built-in browser password managers to dedicated, enterprise-grade password systems that leverage memory-isolated master key loops.
* **Rotate Local DPAPI Master Secrets:** Mandate regular user account password updates linked directly with domain controller validation systems to enforce continuous rollover transformations across local DPAPI master storage boundaries.
* **Employ Robust File System Encryption:** Restrict local administrator capabilities across production user devices to prevent rogue actors from executing registry hive triage extractions (`SAM`/`SYSTEM`) or dumping system configuration memory spaces.
