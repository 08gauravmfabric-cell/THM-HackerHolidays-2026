# 🗺️ Day 05 — Beach Bar

## 📝 Room Information
* **Category:** Boot2Root
* **Vulnerability:** Unsafe YAML Deserialization

---

## 🚀 Attack Chain / Walkthrough
Injected a malicious PyYAML payload into the ordering application to trigger a reverse shell.

![YAML Reverse Shell](./screenshot.png)

---

## 🛠️ Tools Used
* Python 3
* Netcat
