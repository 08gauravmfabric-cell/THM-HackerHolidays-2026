# TryHackMe Hacker Holidays 2026 — Day 4: Packed Light Write-up

## 📝 Executive Summary
Day 4 introduces a complex network forensics and reverse-engineering scenario set inside the network architecture of the Byte Lotus Hotel. In this challenge, an insider threat has deployed an obfuscated, memory-resident malware sample on a receptionist's workstation to capture and leak guest credentials. 

By analyzing a raw network packet capture (`traffic.pcapng`) and auditing a recovered Python artifact (`updates.py`), we reverse-engineer a flawed custom XOR encryption scheme. This analysis allows us to reconstruct a covert exfiltration channel hidden within HTTP cookies and retrieve the target flag without compromising operational security.

---

## 🏗️ Environmental Architecture & Threat Model

### 1. The Threat Vector
The attacker gained low-privilege access to the target endpoint and established persistence by masquerading a malicious Python script as an administrative patch named `updates.py`. 

### 2. The Assets Provided
* `traffic.pcapng`: A promiscuous-mode packet capture recording traffic traversing the internal VLAN segment during the suspected exfiltration window.
* `updates.py`: A weaponized Python automation script containing keylogging functions and a hardcoded, low-entropy cryptographic routine.

---

## 🔍 Detailed Technical Walkthrough

### Step 1: Code Review & Cryptographic Analysis
Static analysis of the recovered file `updates.py` reveals that the script establishes an asynchronous listener using the `pynput.keyboard` module. Every keystroke intercepted from the user interface triggers a callback function designed to package and exfiltrate data.

#### The Code Structure:
```python
import pynput.keyboard
import requests
import base64

p1 = "H0t3lSt@ff0Nly"
p2 = "K3epS3cr3t!"
xor_key = p1 + p2  # Result: H0t3lSt@ff0NlyK3epS3cr3t!

def exfiltrate_char(character):
    # Malicious cryptographic and network logic
    ...
```

#### Explaining the Logic Flaw:
The malware developer intended to cycle through the long string `H0t3lSt@ff0NlyK3epS3cr3t!` using a repeating XOR key index. However, because the character-trapping loop calls `exfiltrate_char()` independently for **each individual keystroke**, the function initializes a fresh execution frame every time. 

* The key index resets to zero (`0`) for every single letter typed.
* Every character is exclusively XORed against the first byte of the key string: **`H`** (Hexadecimal: `0x48`, Binary: `01001000`).
* After the single-byte XOR operation, the byte is Base64 encoded and wrapped into an outbound network packet.

---

### Step 2: Advanced Network Forensics via Wireshark
Opening the `traffic.pcapng` file inside Wireshark reveals dense background noise, including standard corporate HTTP, DNS, and NTP requests. To isolate the malicious heartbeat traffic, we map the script behavior back to network indicators.

1. **Protocol Filtering:** The script initiates connections to an external server on port `8080`. We isolate this socket with the display filter:
   ```text
   tcp.port == 8080 && http
   ```
2. **Identifying the Covert Field:** Inspecting the parsed HTTP packets reveals sequential `GET` requests hitting an external endpoint. The malware blends in by placing its payload inside the `Cookie:` header, specifically under a parameter named `hotel_access_state=`.
3. **Payload Extraction:** By scrolling down the packet list view, we note that each sequential packet contains a single Base64 encoded string character inside this cookie parameter. 
4. **Automating Assembly:** To extract these fragments cleanly, we can use `tshark` (the command-line counterpart to Wireshark) to dump the specific HTTP cookie values into a text document:
   ```bash
   tshark -r traffic.pcapng -Y "http.request" -T fields -e http.cookie | grep "hotel_access_state"
   ```

This provides an ordered list of encoded characters representing the encrypted flag stream.

---

### Step 3: Decryption & Flag Recovery Pipeline
With the raw Base64 data chunks ordered sequentially, we assemble the ciphertext string. Because the encryption scheme degrades into a basic single-byte substitution cipher due to the coding flaw, we use CyberChef to reverse the pipeline safely.

#### CyberChef Recipe Configuration:
1. **Input:** Paste the concatenated string of Base64 chunks extracted from the network capture.
2. **Operations:**
   * **From Base64**: Decodes the transmission envelope back into raw binary bytes.
   * **XOR**: Configure the operation with the Key type set to `UTF-8` and input the character `H` (or set Key type to `Hex` and input `48`).

The text string immediately decodes in the output panel, rendering the structural TryHackMe flag format.

---

## 🏁 Flag Capture
To comply with platform policies regarding walkthrough deployments and to protect the integrity of the active learning platform, the literal flag value is masked below:

* **Flag:** `THM{REDACTED_HIDDEN_FOR_POLICY_COMPLIANCE}`

---

## 🛡️ Strategic Incident Remediation

### 1. Network Layer Controls
* **Egress Filtering:** Implement strict firewall rules restricting internal workstations from initiating direct outbound sessions over non-standard application ports like `8080`. All web requests should flow exclusively through an authenticated enterprise web proxy.
* **Deep Packet Inspection (DPI):** Deploy intrusion detection systems (IDS/IPS) configured with rules to trigger alerts when cookie values or headers show structural hallmarks of automated single-character exfiltration patterns.

### 2. Endpoint Security Architecture
* **API Hooking Restrictions:** Deploy Endpoint Detection and Response (EDR) agents calibrated to flag or block untrusted, unsigned Python or executable binaries that attempt to hook keyboard APIs or call native Windows UI framework bindings.
* **Software Restriction Policies (SRP):** Enforce application whitelisting (AppLocker or WDAC) to block developers' runtime environments like standard Python interpreters on production receptionist systems where they serve no business function.
