# Day 4 — Packed Light

**TryHackMe Hacker Holidays 2026 — The Byte Lotus Hotel**  
**Category:** Forensics / Network  
**Difficulty:** Easy  
**Points:** 60  
**Room Link:** https://tryhackme.com/room/hh-packedlight-02e5330c

---

## Concierge Briefing

Tiny packets. Odd hours. Suspiciously regular.  
Someone’s smuggling out the data equivalent of a hotel towel every night, folded neatly inside traffic that looks ordinary until you decode it.

A short capture from the guest network is all VERA could pull before the connection dropped. Somewhere in that traffic, a quiet little errand is running on a loop, and it isn’t part of any service the hotel actually offers.

---

## Objectives

1. Analyze the provided capture for a covert communication channel.
2. Identify where the exfiltrated data is being hidden and reassemble it.
3. Decode the recovered data and submit the flag.

---

## Walkthrough

### Step 1: Download the Capture
Download the task files from the room. You will get a packet capture file (usually named `traffic.pcapng` or similar).

### Step 2: Open the Capture
Open the file in **Wireshark**.

Useful display filters:
tcp.port == 8080

### Step 3: Identify the Covert Channel
Look for repeating HTTP requests that happen at regular intervals (approximately once per second).

Typical indicators:
- Destination host related to the hotel (example: `byte-lotus-hotel.thm`)
- Port `8080`
- Unusual User-Agent (example: `ByteLotusClient/1.1`)
- Suspicious Cookie or custom header (commonly named something like `hotel_sess_state`)

The data being exfiltrated is hidden inside the **Cookie** (or another custom header).

### Step 4: Extract the Hidden Data
Extract all the cookie/header values from the relevant packets.

You can do this in Wireshark:
- Right-click a packet → Follow → HTTP Stream
- Or use File → Export Packet Dissections

Faster method with `tshark`:

```bash
tshark -r traffic.pcapng -Y "http.cookie" -T fields -e http.cookie****
Step 5: Reassemble the Data
Collect all the pieces of the cookie values in the correct order (they are usually sequential).
Concatenate them into one long string.
Step 6: Decode
The concatenated string is almost always Base64 encoded.
Decode it:

Flag THM{[REDACTED]}
(Flag intentionally redacted in accordance with TryHackMe’s spoiler policy. Solve the room yourself to obtain it.)

Key Takeaways

Covert channels frequently abuse cookies or custom HTTP headers for data exfiltration.
Regular, clock-like beaconing is a strong sign of malicious activity.
Always examine every HTTP header carefully during network forensics.
Tools like tshark make extraction much faster than doing it manually in Wires
THM{[REDACTED]}
(Flag intentionally redacted in accordance with TryHackMe’s spoiler policy. Solve the room yourself to obtain it.)

Key Takeaways

Covert channels frequently abuse cookies or custom HTTP headers for data exfiltration.
Regular, clock-like beaconing is a strong sign of malicious activity.
Always examine every HTTP header carefully during network forensics.
Tools like tshark make extraction much faster than doing it manually in Wires
Happy Hacking!
Part of the Hacker Holidays 2026 series.
Next recommended room: Day 5 — Beach Bar
