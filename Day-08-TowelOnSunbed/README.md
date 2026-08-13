# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 8 (Towel on the Sunbed)

## 📖 Introduction & Scenario Context
Day 8 shifts focus to **Business Logic Vulnerabilities**, **API Abuse**, and **Race Conditions**. The setting takes place at the Byte Lotus Hotel's pool deck, where a digital cryptocurrency rewards app created by "Ponzi" manages sunbed reservations and token claims. 

The rewards app enforces a strict 24-hour cooling-off configuration timer per user claim. However, by analyzing the multi-step transaction process, we uncover an architectural race condition flaw in how the database manages state concurrency. This write-up documents the step-by-step methodology to abuse the API, bypass the time lock restriction, and open the high-value "Whale Vault".

---

## 🛠️ Step 1: Initial Application Mapping & Interaction
We begin by establishing a baseline profile of the web application and monitoring normal transactional API endpoints.

1. **Service Access:** Opened the target virtual machine IP address in a web browser to access Ponzi's custom rewards and sunbed reservation app interface.
2. **Account Creation:** Registered a test guest account and logged in to review the user dashboard interface.
3. **Inspecting Dashboard Metrics:** The landing layout tracks your current token balance (e.g., `0 Ponzi Coins`) and displays an option button labeled **"Claim Daily Crypto Reward"** (valued at `50 Ponzi Coins`).
4. **Vault Requirements:** Noticed an elite terminal area labeled **"Whale Vault"** that requires a minimum threshold balance of **`150 Ponzi Coins`** to open. Under normal operational parameters, a user would need 3 full days of consecutive interaction to unlock this section.

---

## 🔍 Step 2: API Sniffing & Business Logic Analysis
We use an interception proxy to analyze how token balances are written and modified on the backend database.

1. **Proxy Configuration:** Initialized **Burp Suite** and passed all browser interactions directly through the local intercept pipeline.
2. **Capturing the Request Envelope:** Clicked the **"Claim Daily Crypto Reward"** button once. In Burp Suite, look at your **HTTP History** window to locate the specific API transaction endpoint:
   - Target Request Method: `POST`
   - Target End Point: `/api/v1/rewards/claim`
3. **Inspecting the Server State Response:** The server processes the initial claim, updates your token count by `+50`, and updates a column in the database tracking your timestamp context.
4. **Testing Cooldown Logic:** Attempted to push a subsequent request immediately afterward. The backend application blocks the connection and responds with a soft logic alert error payload:
   - HTTP Status: `429 Too Many Requests` or `200 OK` with JSON: `{"success": false, "message": "Cooldown active. Try again in 24 hours."}`

---

## 🚀 Step 3: Engineering the Race Condition (API Abuse)
The application validates *if* a user has claimed a reward, but a vulnerability exists in its concurrency model: it queries the database before the writing operation of the previous session finishes executing completely. We leverage **HTTP/2 Parallel Request Groups** to bypass this check.

1. **Staging Requests:** Locate the valid `/api/v1/rewards/claim` POST query packet inside Burp Suite, right-click, and select **"Send to Repeater"**. Repeat this step 3 to 4 times to generate multiple duplicate tabs.
2. **Creating a Parallel Request Group:**
   - In Burp Repeater, look at the upper panel tabs and select the **Plus (+)** or dropdown menu next to the tabs to create a new group.
   - Choose **"Create request group"** and add all the duplicated reward claim requests into this isolated cluster.
3. **Synchronizing Execution Frameworks:**
   - Change the sending configuration from **"Send requests sequentially"** to **"Send requests in parallel (single-packet attack)"**. This mechanism attempts to collapse the TCP frames so that they hit the target processing socket at the exact same millisecond window.
4. **Triggering the Exploit:** Click the **"Send Group"** button to execute the attack cluster concurrently.

---

## 🔓 Step 4: Vault Access & Flag Retrieval
By hitting the server simultaneously, all concurrent connections check the database *before* the first connection successfully sets the 24-hour cooldown lock state, allowing multiple payouts.

1. **Reviewing Response Output:** Inspect the response metrics inside the group display box. Notice that three requests return a completely successful execution response payload (`"success": true`) instead of triggering a cooldown violation.
2. **Verifying Token Balances:** Refresh your active web browser application interface session. Your balance updates from `50` to **`150 Ponzi Coins`**, successfully tricking the application into validating 3 days' worth of rewards in a single step.
3. **Unlocking the Secret Asset:** Navigate down to the **Whale Vault** terminal area on the front-end layout. 
4. **Extracting the Flag:** Click on the **"Open Vault"** command button. Because your balance configuration explicitly satisfies the application logic rule (`balance >= 150`), the hidden database panel exposes the raw target flag string.

---

## 🏁 Flag Capture
To ensure absolute compliance with TryHackMe walkthrough platform deployment frameworks and solution protection policies, the literal flag text is securely hidden below:

* **Flag:** `THM{REDACTED_BUSINESS_LOGIC_CONCURRENCY_BYPASS}`

---

## 🛡️ Strategic Mitigation Actions
* **Implement Database Transaction Locks:** Ensure that the database operations use ACID-compliant transactions with explicit raw locking strategies. When processing a financial or token payout, wrap the workflow inside a `SELECT ... FOR UPDATE` statement to lock the target user row context until the claim state finishes updating completely.
* **Idempotency Token Implementation:** Force the web front-end client interface to request and append a unique, single-use **Idempotency-Key** header variable token with each request. The backend middleware should intercept, record, and drop any duplicate incoming packets processing the exact same key string value inside a short execution window.
