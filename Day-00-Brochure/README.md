# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 0 (The Brochure)

## 📖 Introduction & Scenario Context
Day 0 serves as the foundational footprinting phase for the Hacker Holidays engagement at the Byte Lotus Hotel. Management published a digital, interactive web brochure (`http://<TARGET_IP>/`) to advertise their premium guest accommodations and automated check-in services. 

Before launching targeted structural attacks, an analyst must perform non-intrusive mapping to catalog open web sockets, uncover hidden web assets, and inspect exposed client-side source materials. This technical write-up documents the step-by-step process of footprinting the target domain and extracting hidden metadata markers to solve the introductory challenges.

---

## 🛠️ Step 1: Network Footprinting & Port Mapping
We begin by establishing a baseline configuration profile of the hotel's public infrastructure boundary to isolate active network interfaces.

1. **Initializing Nmap Scan Sequence:** Launch a comprehensive service and script scanning routine against the target virtual machine's IP address:
   ```bash
   nmap -sV -sC -A -p- --min-rate 5000 <TARGET_IP>
   ```
   *   `-sV`: Detects exact versions running on open network ports.
   *   `-sC`: Runs default Nmap NSE enumeration scripts to flag known vulnerabilities.
   *   `-A`: Enforces aggressive OS identification and traceroute mapping.
2. **Analyzing the Scan Matrix:** Review the terminal execution log output to map listening channels:
   ```text
   PORT   STATE SERVICE VERSION
   80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
   ```
3. **Validating Web Entry Point:** The network scan registers an open HTTP protocol handler on port `80`, confirming a live web server hosting the digital brochure.

---

## 🔍 Step 2: Automated Directory Discovery & Content Enumeration
While the main website structure appears to be a standard brochure interface, web administrators frequently leave legacy code, drafts, or configuration parameters hidden inside unlinked directories.

1. **Configuring the Gobuster Engine:** Execute a web directory brute-forcing script routine using a standard wordlist to look for hidden directory nodes:
   ```bash
   gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -x html,txt,php,bak
   ```
   *   `-u`: Specifies the root destination target web path framework.
   *   `-w`: References the system directory traversal wordlist payload file.
   *   `-x`: Directs the scraper engine to search for specific file type suffixes (`.html`, `.txt`, `.php`, `.bak`).
2. **Reviewing Resource Status Codes:** Monitor the discovery terminal to trace anomalous HTTP `200 OK` or `301 Redirect` execution returns:
   ```text
   /index.html           (Status: 200)
   /images               (Status: 301)
   /assets               (Status: 301)
   /secret_notes.txt     (Status: 200)
   ```
3. **Inspecting Discovered Assets:** Use the terminal utility `curl` to grab the text headers of any raw file assets found during the sweep:
   ```bash
   curl -X GET http://<TARGET_IP>/secret_notes.txt
   ```

---

## 🚀 Step 3: Client-Side Source Auditing & Code Inspection
We pivot our focus to analyzing the structural design components of the main application platform layout files (`index.html`) to look for misconfigured metadata fields.

1. **Target Interaction:** Open your local browser environment and navigate to the brochure web portal: `http://<TARGET_IP>/`.
2. **Opening the Inspector Portal:** Right-click an empty workspace field inside the website canvas area and select **View Page Source** (or press the `F12` Developer Console macro).
3. **Searching for Hidden Developer Artifacts:** Scroll through the active structural layout tags. Search specifically for hidden HTML block comments (`<!-- comment -->`) or inline JavaScript entries.
4. **Locating the Configuration Leak:** Identify an isolated developer notation block appended to a specific hotel service element or background script link:
   ```html
   <!-- TODO: Remove this debug flag reference before production deployment -> THM{...} -->
   ```
5. **String Extraction:** Isolate the hidden parameter value embedded cleanly within the comment text string.

---

## 🏁 Flag Capture
To explicitly maintain absolute compliance with TryHackMe walkthrough platform deployment frameworks and solution protection policies, the literal flag text string is securely hidden below:

* **Flag:** `THM{REDACTED_INTRODUCTORY_BROCHURE_FOOTPRINTING_SUCCESS}`

---

## 🛡️ Strategic Mitigation Actions
* **Disable Directory Listing & Information Disclosure:** Configure the Apache web server runtime policies to restrict anonymous resource mapping across asset boundaries. Disable the `Indexes` option inside the `httpd.conf` configuration profile to block scanners from mapping folder directories.
* **Purge Production Comments:** Enforce a strict code compilation baseline pipeline that automatically sanitizes development build files. This strips out all engineering notes, raw comment tags, and testing identifiers before staging application assets on public-facing internet nodes.
* **Expose Minimal System Headers:** Adjust global web framework settings (`ServerTokens Prod` and `ServerSignature Off`) to obscure the explicit kernel versions or server software types running on port `80`, minimizing the intelligence accessible to an attacker.
