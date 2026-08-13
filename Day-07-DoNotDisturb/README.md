# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 7 (Do Not Disturb)

## 📖 Introduction & Scenario Context
Day 7 returns to web application pentesting at the Byte Lotus Hotel. The focus shifts to an online room service and guest accommodation tracking dashboard. Security teams flagged potential data leakage vectors within the internal guest request processing features. 

This technical write-up documents the step-by-step discovery, inspection, and exploitation of an **Insecure Direct Object Reference (IDOR)** vulnerability that grants unauthorized access to administrative bookings, resulting in flag retrieval.

---

## 🛠️ Step 1: Initial Reconnaissance & Target Mapping
We begin by establishing a baseline configuration profile of the web application framework to locate vulnerable interaction points.

1. **Service Verification:** Navigated to the deployment machine's IP address on the designated web service port to access the guest portal interface.
2. **Account Provisioning:** Interacted with the frontend application to create a low-privilege guest tester account or log in with provided basic room credentials.
3. **Intercept Setup:** Configured an interception proxy like **Burp Suite** or opened the browser's native **Developer Tools (F12)** network observation panel to record all outbound asynchronous web requests (`XHR/Fetch`).
4. **Analyzing Request Parameters:** Navigated to the personal dashboard tracking your room's active request queue (e.g., requesting extra towels, room service status, or checkout profiles).

---

## 🔍 Step 2: Spotting the Logic Flaw & Testing for IDOR
By analyzing how the backend database references individual objects, we can test the system's authorization boundaries.

1. **Isolating the Resource Handle:** Observed the URL structure or API parameters when loading a specific service request record. The application structures its resource paths using numeric identifier tags:
   - Target Request URI: `http://<TARGET_IP>/api/requests/view?id=1042`
2. **Testing Access Controls:** Copied the exact request headers into the Burp Suite **Repeater** module. 
3. **Manipulating the Identifier:** Changed the resource parameter value dynamically (`id=1041`, `id=1040`, `id=1`) to observe how the backend handles variations in input.
4. **Confirming the Vulnerability:** Instead of returning a `403 Forbidden` or a redirect block when querying an item belonging to a completely different user profile, the server processes the state change cleanly and returns the full JSON object payload for the targeted record. This confirms a clear lack of object-level authorization checking.

---

## 🚀 Step 3: Automating Resource Enumeration & Administrative Elevation
To locate administrative service flags hidden within the multi-tenant framework database without manually guessing IDs, we automate the discovery pipeline.

1. **Setting up the Intruder Engine:** Sent the validated repeating request block directly to the Burp Suite **Intruder** tool (or built a simple custom Python threading loop script).
2. **Defining the Injection Point:** Marked the target variable parameter on the `id=` value block as our payload injection coordinate.
3. **Configuring the Number Range Sequence:** 
   - Set the payload type to **Numbers**.
   - Established the generation sequence bounds (e.g., from `1` to `2000`, using a step parameter increment value of `1`).
4. **Executing the Collection Sweep:** Launched the enumeration sweep and closely evaluated the resulting transaction metrics table.
5. **Analyzing Structural Divergence:** Sorted the output row records dynamically by response length (`Length`) or specific HTTP status indicators to find administrative room numbers or system maintenance handles.

---

## 🔓 Step 4: Decoding the Admin Payload & Flag Retrieval
A highly specific object ID reveals a specialized hotel maintenance or administration booking record that does not correspond to a normal guest profile.

1. **Inspecting the Vulnerable Response:** Opened the response data associated with the discovered unique identifier index block.
2. **Reviewing Data Parameters:** The returned JSON or HTML structure displays internal system metadata, room service configuration overrides, or special administrative text logs.
3. **Extracting the Flag String:** Read down the configuration values to discover the target flag string formatted and saved directly inside the parameter fields.

---

## 🏁 Flag Capture
To strictly maintain compliance with TryHackMe walkthrough formatting policies, the actual key string value is securely redacted below:

* **Flag:** `THM{REDACTED_INSECURE_DIRECT_OBJECT_REFERENCE_BYPASS}`

---

## 🛡️ Strategic Mitigation Actions
* **Implement Object-Level Access Tokens:** The backend web framework must never rely exclusively on a client-supplied numeric index parameter to fetch data rows. Validate the user's active session cookie token (`JWT` or session ID) against a database authorization access-control list (ACL) before serving the resource object.
* **Adopt Non-Sequential Identifiers:** Replace sequential auto-incrementing integer primary database keys with cryptographic Globally Unique Identifiers (**UUIDv4**). Changing paths to string variants like `/api/requests/view?id=f81d4fae-7dec-11d0-a765-00a0c91e6bf6` completely breaks predictable parameter guessing attacks.
