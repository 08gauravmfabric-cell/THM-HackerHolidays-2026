# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 6 (Overheard at Breakfast)

## 📖 Introduction & Scenario Context
Day 6 shifts focus away from active infrastructure exploitation into **Open-Source Intelligence (OSINT)** and **Social Media Hashing**. The narrative unfolds at the breakfast terrace of the Byte Lotus Hotel, where a guest captures a raw screenshot of a text message conversation between two individuals named "Ponzi" and "Lambo". 

One of the users has claims of completely wiping their presence from the internet. This forensic write-up documents the comprehensive, step-by-step metadata verification, text extraction, profile profiling via Gravatar, and cryptographic string analysis required to discover the hidden target account and capture the flag.

---

## 🛠️ Step 1: Initial Artifact Analysis & Metadata Verification
We begin the investigation by downloading the provided zipped task file and confirming the baseline integrity of the image payload.

1. **Extracting the Archive:** Unzip the downloaded room file locally on your forensic analyst machine:
   ```bash
   unzip overheard_at_breakfast.zip
   ```
2. **Reviewing Image Metadata:** Run `ExifTool` against the extracted PNG screenshot file to check for hidden geolocation data, modification timestamps, or structural software tags left behind during creation:
   ```bash
   exiftool conversation_leak.png
   ```
3. **Steganographic Check:** Rule out standard low-level data hiding techniques by running `zsteg` to review LSB (Least Significant Bit) channels for buried hexadecimal or string sequences:
   ```bash
   zsteg conversation_leak.png
   ```

---

## 🔍 Step 2: Manual Conversation Inspection & OCR Extraction
After confirming that no file-level hidden elements are present, we look closely at the visible conversational exchange displayed inside the image canvas.

1. **Textual Content Reading:** Reviewing the text exchange reveals Lambo discussing a specific, highly identifying communication artifact: a unique personal corporate email address.
2. **Extracting Text cleanly:** To copy the text from the pixel matrix without manual typing errors, use an Optical Character Recognition (**OCR**) tool or utility:
   ```bash
   tesseract conversation_leak.png stdout
   ```
3. **Isolating the Indicator of Compromise (IoC):** Jot down the target email address exactly as printed in the conversation thread, ensuring all characters match casing guidelines:
   - Target Identified: `lambo_investor_99@bytelotus.thm` *(Example target format structure)*

---

## 🌐 Step 3: Platform Identification & Social Media Hashing
During the chat logs, the user references a universal profile platform used to unify public avatars, descriptions, and verified online social identities. This is identified as **Gravatar**.

### Understanding Gravatar's Architecture
Gravatar does not allow global profile lookups using plaintext email addresses via standard search bars. Instead, it utilizes an address mapping system dependent on **cryptographic hashing**.
* To query the profile API database endpoint, the email address must be converted into a single **MD5 string hash**.
* **Pre-processing Requirement:** The email string must be completely lowercase, and any trailing spaces must be cut before compiling the hash.

### Generating the Compliant MD5 Hash
Run a pipeline bash script sequence to normalize the email handle and calculate its true cryptographic hash output:
```bash
echo -n "lambo_investor_99@bytelotus.thm" | tr '[:upper:]' '[:lower:]' | md5sum
```
* `-n`: Prevents a hidden trailing newline character `\n` from corrupting the input block.
* `tr '[:upper:]' '[:lower:]'`: Strictly guarantees compliance with lowercase format requirements.

Take the resulting output string (e.g., `4a3b2c...`) to formulate the public directory tracking link.

---

## 🚀 Step 4: Tracking the Hidden Account Profile
With the finalized MD5 identifier generated, we execute our open-source intelligence lookup pipeline against the primary live infrastructure directory.

1. **Crafting the Endpoint Path:** Construct the lookup URL template by appending the custom hash directly to the master directory tracking path:
   - Path Template: `https://gravatar.com<YOUR_GENERATED_MD5_HASH>.json`
2. **Querying the Directory:** Submit the connection request via browser navigation or use `curl` to capture the profile details cleanly from your command line interface:
   ```bash
   curl -s https://gravatar.com4a3b2cb9e8...json | jq
   ```
3. **Analyzing Content Output:** The lookup returns an authenticated, active profile mapping structure. Open the `profileUrl` or look at the `aboutMe` text bio block.
4. **Locating the Hidden Payload:** Inside the user's bio profile data, we observe an anomalies string sequence structured precisely inside a Base64 encoding configuration envelope.

---

## 🔓 Step 5: Decoding the Target Payload
We push the discovered target configuration data string back into our local terminal framework to parse out the underlying flag values safely.

1. **Isolate Payload Block:** Copy the raw string segment retrieved out of the user's bio entry field.
2. **Execute Terminal Reversal:** Pass the payload string directly into the base64 operating framework utility:
   ```bash
   echo "VUhNZ2RHOXpaV0YwWlhKekxuaDBiVzB9..." | base64 -d
   ```

The operational interface decodes the base data, immediately resolving into the structured format block expected by the assessment engine.

---

## 🏁 Flag Capture
To explicitly uphold TryHackMe's anti-cheating, solution-protection, and deployment guidelines, literal flag outputs are completely omitted below:

* **Flag:** `THM{REDACTED_GRAVATAR_OSINT_IDENTITY_DISCOVERED}`

---

## 🛡️ Strategic Mitigation Actions
* **Enforce Email Hashing Privacy:** While MD5 hashes obscure raw email addresses, they are incredibly susceptible to brute-force or pre-computed rainbow table lookups. Sensitive corporate accounts should never be mapped to public directory trackers like Gravatar.
* **Control Corporate Digital Footprints:** Security teams must implement continuous OSINT tracking and external asset audits. This ensures employees do not create public developer profiles linking private internal operational domains to outside web nodes.
