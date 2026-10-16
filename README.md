# 🔐 Injectics CTF — Complete Web Application Penetration Testing Write-up

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme" alt="TryHackMe"/>
  <img src="https://img.shields.io/badge/Category-Web%20Application%20Security-blue?style=for-the-badge" alt="Web Application Security"/>
  <img src="https://img.shields.io/badge/Status-Completed-success?style=for-the-badge" alt="Completed"/>
  <img src="https://img.shields.io/badge/Focus-Penetration%20Testing-orange?style=for-the-badge" alt="Penetration Testing"/>
</p>

<p align="center">
  <b>Hands-on Web Application Security Assessment | SQL Injection | SSTI | RCE | Linux Enumeration</b>
</p>

---

## 📌 Project Overview

This repository contains my detailed penetration testing documentation for the **Injectics Capture The Flag (CTF) room on TryHackMe**.

The assessment focused on identifying and validating security vulnerabilities in a PHP-based web application backed by a MySQL database. The engagement covered the complete testing workflow, beginning with network reconnaissance and web enumeration and progressing through authentication testing, authorization analysis, server-side template injection, controlled remote code execution, and flag retrieval.

During this lab, I examined how multiple application security weaknesses can combine to compromise an entire web application. The assessment demonstrated the risks of exposing sensitive files, relying on client-side validation, constructing SQL queries unsafely, implementing insufficient access controls, and evaluating user-controlled input as template code.

The final objective was achieved by recovering both challenge flags and documenting the findings in a structured penetration testing report with supporting screenshots, impact assessments, reproduction steps, and remediation recommendations.

This repository is intended to demonstrate my practical understanding of web application penetration testing, vulnerability analysis, exploitation validation, Linux-based security testing, and technical report writing.

## 👨‍💻 About the Researcher

**Name:** Nishant Saini  
**Background:** BCA Graduate  
**Current Learning Focus:** Cybersecurity and Ethical Hacking  
**Specialization Interests:** Web Application Security, Penetration Testing, Vulnerability Assessment, Linux Security, and Offensive Security Fundamentals.

I completed this assessment as part of my hands-on cybersecurity learning journey. My objective was not only to retrieve the flags but also to understand the root causes of the vulnerabilities, their security implications, how the issues could be reproduced in an authorized lab, and how developers could prevent similar weaknesses in production applications.

## 🎯 Lab Information

| Category | Details |
|---|---|
| Lab Name | Injectics |
| Platform | TryHackMe |
| Lab Type | Capture The Flag (CTF) |
| Target Application | Injectics 2024 |
| Application Technology | PHP, JavaScript, jQuery, Bootstrap |
| Database | MySQL |
| Target Operating System | Ubuntu |
| Web Server | Apache HTTP Server 2.4.41 |
| Initial Access Vector | Web application vulnerabilities |
| Main Vulnerabilities | SQL Injection, SSTI, Broken Access Control |
| Post-Exploitation Context | `www-data` |
| Assessment Period | 15–16 October 2026 |
| Result | Both CTF flags captured |
| Report Format | PDF penetration testing report |

> **Scope notice:** The IP addresses used during this assessment belonged to temporary TryHackMe lab deployments. They are not permanent target addresses and should not be treated as live systems.

## 📖 Table of Contents

- [Project Overview](#-project-overview)
- [Lab Information](#-lab-information)
- [Objectives](#-objectives)
- [Scope and Rules of Engagement](#-scope-and-rules-of-engagement)
- [Testing Environment](#-testing-environment)
- [Tools and Technologies](#️-tools-and-technologies)
- [Methodology](#-methodology)
- [Phase 1 — Reconnaissance](#-phase-1--reconnaissance)
- [Phase 2 — Web Enumeration](#-phase-2--web-enumeration)
- [Phase 3 — Information Disclosure](#-phase-3--information-disclosure)
- [Phase 4 — SQL Injection](#-phase-4--sql-injection)
- [Phase 5 — Authorization Testing](#-phase-5--authorization-testing)
- [Phase 6 — Administrative Access](#-phase-6--administrative-access)
- [Phase 7 — Server-Side Template Injection](#-phase-7--server-side-template-injection)
- [Phase 8 — Remote Code Execution](#-phase-8--remote-code-execution)
- [Phase 9 — Post-Exploitation](#-phase-9--post-exploitation)
- [Complete Attack Chain](#-complete-attack-chain)
- [Vulnerability Assessment](#-vulnerability-assessment)
- [Detailed Findings](#-detailed-findings)
- [Flags Captured](#-flags-captured)
- [Remediation Roadmap](#-remediation-roadmap)
- [Key Technical Learnings](#-key-technical-learnings)
- [Challenges and Problem-Solving](#-challenges-and-problem-solving)
- [Evidence and Documentation](#-evidence-and-documentation)
- [Conclusion](#-conclusion)
- [Disclaimer](#️-disclaimer)
- [Connect With Me](#-connect-with-me)

---

## 🎯 Objectives

The primary objectives of this engagement were:

1. Identify open ports and exposed services on the target.
2. Understand the web application's structure and technologies.
3. Discover hidden directories, administrative endpoints, and potentially sensitive files.
4. Review HTML and JavaScript source code for insecure implementation patterns.
5. Investigate exposed credentials and authentication weaknesses.
6. Test the login functionality for SQL injection.
7. Evaluate whether client-side validation could be bypassed.
8. Test authorization controls on leaderboard modification functionality.
9. Identify template expression evaluation in profile updates.
10. Validate the impact of server-side template injection.
11. Demonstrate remote command execution in the authorized lab.
12. Enumerate the web server's accessible files and recover both flags.
13. Document each finding with severity, impact, evidence, and remediation guidance.

The assessment followed a practical penetration testing workflow, emphasizing evidence-based validation rather than relying exclusively on automated scanner output.

## 🔒 Scope and Rules of Engagement

The assessment was restricted to the Injectics TryHackMe environment.

### In scope

- Network and service enumeration of the assigned lab target.
- HTTP service and web application inspection.
- Directory and file discovery.
- Authentication and authorization testing.
- SQL injection testing.
- Template injection testing.
- Controlled command execution to demonstrate vulnerability impact.
- Limited post-exploitation enumeration to locate challenge flags.
- Security recommendations based on the observed weaknesses.

### Out of scope

- Testing unrelated internet-facing systems.
- Attacking third-party infrastructure.
- Persistence outside the lab objectives.
- Privilege escalation beyond the demonstrated `www-data` account.
- Destructive testing against production services.
- Accessing data unrelated to the CTF objectives.

The engagement was performed for educational purposes within an authorized training environment.

## 💻 Testing Environment

The testing environment consisted of a Kali Linux workstation and the temporary Injectics target deployed by TryHackMe.

### Attacker environment

- Operating System: Kali Linux
- Network interface used for the lab: `tun0`
- Tools: Nmap, Gobuster, Burp Suite, Netcat, Python HTTP server, and Firefox.
- Testing approach: Black-box testing followed by review of accessible application source and scripts.

### Target environment

- Operating System: Ubuntu
- Web server: Apache HTTP Server 2.4.41
- Backend language: PHP
- Database: MySQL
- Frontend technologies: HTML, JavaScript, jQuery, and Bootstrap
- Exposed services: SSH and HTTP

The report recorded two different target addresses because the room was deployed in separate sessions. The reconnaissance and exploitation activities therefore took place against different temporary instances of the same lab application.

## 🛠️ Tools and Technologies

| Tool | Purpose | How It Was Used |
|---|---|---|
| Nmap | Network scanning | Identified open ports and running services |
| Gobuster | Directory enumeration | Discovered web paths and exposed endpoints |
| Burp Suite Proxy | HTTP interception | Captured requests generated by the application |
| Burp Suite Repeater | Request manipulation | Tested login and leaderboard behavior |
| Firefox | Browser inspection | Examined application pages and responses |
| View Source | Source-code review | Identified frontend validation and hidden links |
| Netcat | Network listener | Received the lab reverse-shell connection |
| Python HTTP Server | HTTP payload delivery | Served a controlled test payload from the attacker machine |
| Kali Linux | Testing workstation | Provided the penetration testing environment |
| Linux shell utilities | Post-exploitation enumeration | Examined the accessible web root and file permissions |

### Core technologies explored

- HTTP request and response handling
- PHP application behavior
- MySQL query construction
- JavaScript-based form validation
- Session cookies
- SQL injection
- Server-Side Template Injection
- Remote Code Execution
- Linux file ownership and permissions
- Web server security configuration

---

## 🔎 Methodology

The assessment followed a phased methodology similar to a practical web application penetration test.

### Phase 1: Reconnaissance

Identify the target's exposed services, web server, application technologies, and initial attack surface.

### Phase 2: Enumeration

Discover directories, administrative endpoints, publicly accessible files, and useful information disclosed by the application.

### Phase 3: Vulnerability Identification

Inspect login handling, frontend validation, access control, profile updates, and exposed administrative interfaces.

### Phase 4: Controlled Validation

Use intercepted HTTP requests and harmless test inputs to confirm whether suspected vulnerabilities could be reproduced.

### Phase 5: Exploitation

Validate the SQL injection and template injection weaknesses and demonstrate their potential impact within the authorized lab.

### Phase 6: Post-Exploitation

Determine the access level obtained, inspect relevant files, and retrieve the required CTF flags.

### Phase 7: Reporting

Record the vulnerabilities, severity, evidence, business impact, and remediation steps.

---

## 🌐 Phase 1 — Reconnaissance

The first phase focused on understanding the target's network exposure and identifying services that could provide an entry point into the application.

### Service discovery

A full TCP port scan with service detection identified two open ports.

| Port | Service | Identified Details |
|---|---|---|
| 22/TCP | SSH | OpenSSH 8.2p1 on Ubuntu |
| 80/TCP | HTTP | Apache HTTP Server 2.4.41 on Ubuntu |

The HTTP service exposed the Injectics web application, while SSH was also available on the host.

The initial inspection also identified a PHP session cookie that did not include the `HttpOnly` attribute.

### Reconnaissance observations

The following details were recorded:

- The target was running an Ubuntu-based web server.
- Apache served a PHP-based web application.
- The public homepage displayed a leaderboard.
- The application exposed a login page.
- A separate administrative login entry point was present.
- The source code contained information useful for understanding the application structure.

### SSH testing

The usernames `dev` and `john` were tested during the assessment. Both attempts were rejected because the SSH service required public-key authentication.

SSH access was not pursued further because it was not necessary to achieve the lab objectives.

**Outcome:** The HTTP application became the primary focus for further testing.

---

## 📂 Phase 2 — Web Enumeration

After identifying the web service, I enumerated application paths and inspected the website's source code.

Directory enumeration revealed multiple application directories and a publicly reachable database administration interface.

### Important paths discovered

| Path | Observation | Security Relevance |
|---|---|---|
| `/flags/` | Returned a restricted response | Indicated the presence of a sensitive directory |
| `/css/` | Static assets | Revealed application structure |
| `/js/` | JavaScript assets | Provided scripts for client-side analysis |
| `/javascript/` | JavaScript-related directory | Additional application information |
| `/vendor/` | Third-party dependencies | Revealed application components |
| `/phpmyadmin/` | phpMyAdmin login page | Exposed a database management interface |
| `/mail.log` | Publicly readable log file | Disclosed sensitive information |
| `/adminLogin007.php` | Administrative login page | Exposed a separate admin entry point |

The `/adminLogin007.php` endpoint was identified by inspecting the public login page's source code.

### Why enumeration mattered

Directory enumeration provided information that was not immediately visible through normal navigation. It exposed additional application components, helped identify the administrative interface, and highlighted the importance of checking whether logs and other non-public files were accidentally served by the web server.

A discovered path does not automatically represent an exploitable vulnerability. Its security impact depends on what the server exposes, how authentication is enforced, and whether the associated functionality is properly protected.

**Outcome:** Enumeration established several leads that were investigated during the vulnerability assessment.

---

## 📨 Phase 3 — Information Disclosure

One of the most significant findings was the publicly accessible `mail.log` file.

The file contained internal application information, including default credentials for the `superadmin` and `dev` accounts. It also described a service that restored default credentials under certain conditions.

### Security impact

Exposed credentials can allow an attacker to access accounts without exploiting a software vulnerability in the authentication mechanism itself.

In this lab, the disclosed credentials did not authenticate successfully through the standard login page, but they were subsequently accepted by the separate administrative login endpoint.

This demonstrated how credential exposure and inconsistent authentication paths can combine to create a critical weakness.

### Additional information disclosure

Other observations included:

- Developer information in HTML comments.
- Details about internal application files.
- Apache version information in server responses.
- Runtime information rendered by the dashboard.
- Messages revealing the operation of an internal service.
- A PHP session cookie without the `HttpOnly` attribute.

Individually, some of these disclosures were relatively minor. Together, they reduced the amount of reconnaissance required to understand the target.

### Remediation

- Remove internal logs from the public web root.
- Restrict access to sensitive file extensions.
- Rotate exposed credentials immediately.
- Remove hardcoded account-creation and credential-restoration logic.
- Avoid including internal details in HTML comments and error messages.
- Configure production HTTP headers to minimize unnecessary information disclosure.

---

## 💉 Phase 4 — SQL Injection

SQL injection was one of the critical vulnerabilities discovered during the engagement.

### Initial investigation

The login functionality submitted an HTTP request to `functions.php`, where the login operation was handled.

Review of the frontend JavaScript revealed a keyword blacklist intended to block potentially dangerous input. However, the restriction was implemented on the client side and did not provide reliable protection against directly submitted HTTP requests.

### Validation process

Burp Suite was used to capture and inspect the application's login request.

The request was then examined in Repeater to determine whether manipulating the username parameter changed the application's authentication behavior.

The assessment demonstrated that the server handled certain specially crafted inputs differently, revealing an incomplete filtering approach and unsafe SQL query construction.

### Result

The authentication check was bypassed, and the application returned a successful login response identifying the `dev` account.

This confirmed that an unauthenticated user could obtain an authenticated application session without supplying the account's legitimate password.

### Why the vulnerability existed

The underlying issue was the construction of a database query using untrusted input without reliable parameter binding.

Filtering a small set of SQL keywords cannot provide robust protection because SQL syntax has multiple operators, expressions, encodings, and valid representations.

### Potential impact

- Authentication bypass.
- Unauthorized access to user accounts.
- Exposure of information stored in the database.
- Potential database modification, depending on the affected query and database permissions.
- Increased risk to administrative functionality if the same weakness affects privileged accounts.

### Recommended remediation

Use prepared statements and parameterized queries throughout the application.

For PHP, database access should use PDO or MySQLi with bound parameters. Database accounts should follow the principle of least privilege, and authentication errors should avoid disclosing unnecessary information.

**Outcome:** The login SQL injection was confirmed and rated Critical.

---

## 🧪 Phase 5 — Client-Side Validation and Authorization Testing

The next stage examined whether the application's validation and access control could be bypassed by modifying HTTP requests directly.

### Client-side validation weakness

The frontend JavaScript checked user input before submitting the login form. Because this check ran in the browser, it could be bypassed by sending a request directly to the server.

This is a common web application security mistake: client-side validation can improve usability, but it must never be treated as the primary security boundary.

**Security principle:** All security-critical validation must be enforced on the server.

### Leaderboard manipulation

The application included a feature for editing leaderboard values.

By intercepting the relevant POST request in Burp Suite, I demonstrated that the application accepted modified leaderboard values without sufficient authorization checks.

During testing, a leaderboard value was changed from 22 to 25, and the modified value appeared in the public leaderboard.

### Why this matters

The issue was not merely that a user could edit a number. The server failed to adequately establish whether the authenticated user had permission to perform the operation.

The application also needed stronger validation of the submitted values.

### Potential impact

- Unauthorized changes to published leaderboard data.
- Loss of data integrity.
- Manipulation of records displayed to other users.
- Abuse of application functions intended for privileged users.

### Recommended remediation

- Verify authorization on the server for every state-changing request.
- Enforce role-based access controls.
- Validate numeric fields and their permitted ranges.
- Use CSRF protections for relevant state-changing actions.
- Log administrative changes and identify the user responsible.
- Keep client-side validation as an additional usability layer rather than a security control.

**Outcome:** The assessment confirmed an authorization and input-validation weakness rated High.

---

## 🔑 Phase 6 — Administrative Access

The public login page contained a link to a separate administrative login page.

Source-code inspection revealed the endpoint `/adminLogin007.php`.

The existence of a separate administrative login is not inherently a vulnerability. The security concern was the reliance on an obscure URL rather than strong, consistently enforced access controls.

### What was demonstrated

The publicly readable mail log disclosed default credentials. Those credentials failed on the standard login page but were accepted at the administrative login endpoint.

The resulting session provided access to the admin dashboard, where the first CTF flag was displayed.

### Security lessons

- Hidden URLs do not replace authentication.
- Administrative functions require explicit authorization.
- Credentials must not be exposed through public files.
- Separate authentication paths must enforce equivalent security controls.
- Administrative accounts should use strong credentials and multi-factor authentication.

### Recommended remediation

Restrict administrative access to trusted networks or a VPN where appropriate, implement MFA, enforce rate limiting, and ensure that every administrative operation performs server-side role checks.

**Outcome:** Administrative access was achieved in the authorized lab, and the first flag was recovered.

---

## 🧩 Phase 7 — Server-Side Template Injection

Server-Side Template Injection (SSTI) was the most significant technical vulnerability investigated during this assessment.

The issue affected the profile update functionality, specifically the First Name field.

### What is SSTI?

SSTI occurs when an application treats user-controlled input as template source code rather than ordinary data.

Template engines are designed to evaluate expressions and generate dynamic output. If untrusted input is evaluated as template code, an attacker may be able to execute unintended expressions. Depending on the engine, its configuration, and available functions, the consequences can extend to server-side command execution.

### Testing process

I submitted a harmless mathematical expression through the profile update functionality.

The application evaluated the expression and displayed its result on the dashboard rather than displaying the original input as text.

This confirmed that the profile field was being interpreted by a server-side template mechanism.

Further controlled testing demonstrated that the template functionality could invoke a PHP command-execution function. The command output identified the running web-server account as `www-data`.

### Why this was critical

Profile names should be treated as data. They should not be passed into a template engine as executable template source.

When an application evaluates user-controlled template expressions, the vulnerability may cross the boundary between ordinary web input and server-side execution.

### Potential impact

- Execution of commands under the web server's account.
- Reading application source files accessible to that account.
- Exposure of configuration information and potentially sensitive data.
- Access to files within the account's permissions.
- A foothold for additional attacks if other weaknesses are present.

The actual impact depends on the privileges of the web server process, filesystem permissions, network restrictions, and the template engine's configuration.

### Recommended remediation

- Never compile or evaluate user-controlled strings as template source.
- Pass user input into templates as ordinary variables.
- Enable contextual output escaping.
- Restrict dangerous functions and callbacks.
- Apply appropriate sandboxing to template engines.
- Run application services with minimum necessary privileges.
- Restrict unnecessary outbound network connections.
- Add regression tests for template injection.

**Outcome:** SSTI was confirmed and rated Critical because it led to remote command execution.

---

## 🖥️ Phase 8 — Remote Code Execution

Following the confirmation of SSTI, the assessment investigated whether the vulnerability could provide a working foothold on the target.

### Controlled validation

The testing process demonstrated command execution as the `www-data` user.

A controlled payload was then served from the testing workstation using Python's HTTP server. The target retrieved the payload, and a listener received the resulting reverse-shell connection.

The resulting shell operated as the web server account within the lab.

### Access level obtained

The session provided access to resources permitted to the `www-data` user.

This was sufficient to enumerate the application web root and retrieve the remaining challenge flag.

### Important distinction

Remote code execution does not automatically mean that an attacker has root access.

In this assessment, the confirmed execution context was `www-data`. Privilege escalation beyond that account was not attempted because the room objective had been achieved and the engagement scope was limited.

### Security implications

Successful command execution can expose the application to consequences beyond the original web request, including file access, unauthorized modification of accessible resources, and additional compromise when permissions or network controls are weak.

### Recommended remediation

Remove the root cause of SSTI rather than relying on a WAF as the primary defense. Additionally, enforce least privilege, restrict filesystem write access, monitor unexpected outbound connections, and alert on unusual child processes launched by the web server.

**Outcome:** Controlled remote command execution was demonstrated within the authorized environment.

---

## 🐧 Phase 9 — Post-Exploitation and Linux Enumeration

After obtaining the `www-data` shell, I performed limited enumeration to identify the files relevant to the lab objectives.

### Filesystem observations

The web root contained application files with insecure ownership and permissions.

The assessment also identified a leftover Vim swap file named `.conn.php.swp` and a writable service script associated with the restoration of default credentials.

The contents of the swap file were not retrieved during the assessment, so the possibility of database credential disclosure was documented as a risk rather than a confirmed disclosure.

### Why file permissions matter

Web applications frequently require read access to source files and controlled write access to specific data directories. Granting broad write permissions to application code can allow an attacker who compromises the web process to modify scripts or influence future application behavior.

Backup files, editor swap files, logs, and configuration files should not be stored in publicly accessible directories.

### Recommended remediation

- Remove swap files, backups, and temporary files from deployment directories.
- Move sensitive configuration files outside the public web root.
- Ensure application code is not writable by the web server account.
- Remove unnecessary group-write permissions.
- Run maintenance services using dedicated, least-privileged accounts.
- Audit file ownership and permissions during deployment.

**Outcome:** The second flag was recovered, and additional filesystem security weaknesses were documented.

---

## 🔗 Complete Attack Chain

The assessment demonstrated how multiple independent weaknesses can combine to compromise an application.

| Stage | Action | Result |
|---|---|---|
| 1 | Network reconnaissance | Identified SSH and HTTP services |
| 2 | Directory enumeration | Discovered exposed paths and phpMyAdmin |
| 3 | Information disclosure | Retrieved default credentials from `mail.log` |
| 4 | SQL injection | Bypassed the standard login mechanism |
| 5 | Authorization testing | Modified leaderboard values |
| 6 | Administrative authentication | Accessed the admin dashboard |
| 7 | Template injection testing | Confirmed server-side expression evaluation |
| 8 | Command execution | Executed a command as `www-data` |
| 9 | Controlled shell access | Obtained a reverse shell in the lab |
| 10 | Filesystem enumeration | Located and retrieved the second flag |

### Attack chain analysis

The most important lesson from this assessment is that a successful compromise did not depend on a single isolated mistake.

Exposed credentials provided a route to privileged application functionality. SQL injection demonstrated that the authentication layer could also be bypassed. Weak authorization controls enabled modification of protected data, while SSTI created a path from application input to server-side command execution.

The combination significantly increased the overall impact.

This is why penetration testing should assess the relationships between vulnerabilities rather than evaluate each finding in isolation.

---

## 🚨 Vulnerability Assessment Summary

Nine findings were documented in the final report.

| ID | Finding | Severity |
|---|---|---|
| F-01 | Default credentials exposed through a public log file | Critical |
| F-02 | SQL injection in login functionality | Critical |
| F-03 | Client-side-only validation | Medium |
| F-04 | Insufficient protection of the administrative login | Medium |
| F-05 | Unauthorized leaderboard modification | High |
| F-06 | SSTI resulting in remote code execution | Critical |
| F-07 | Publicly reachable phpMyAdmin and directory information | Medium |
| F-08 | Insecure file permissions and leftover swap file | Medium |
| F-09 | Information disclosure and session cookie hardening gaps | Low |

**Severity note:** These ratings reflect the assessment documented for this lab. Production severity should be determined using the affected system's context, exploitability, business impact, exposure, and an appropriate risk-rating methodology.

## 🔬 Detailed Findings

### F-01 — Exposed Default Credentials

**Severity:** Critical

**Affected component:** `/mail.log`

**Description:** A publicly readable log file contained internal information, including default credentials for privileged and developer accounts.

**Impact:** An attacker could obtain credentials without prior authentication. In the lab, the credentials enabled access through the separate administrative login.

**Root cause:** Sensitive operational information was stored in a publicly served location, and default credential restoration was part of the application design.

**Remediation:**
- Remove the exposed log file.
- Rotate all affected credentials.
- Remove hardcoded defaults.
- Prevent direct access to sensitive files.
- Replace persistent default accounts with a secure, one-time initialization process.

### F-02 — SQL Injection in Authentication

**Severity:** Critical

**Affected component:** `functions.php`, login operation

**Description:** User-controlled login input was incorporated into SQL query construction without reliable parameter binding.

**Impact:** Authentication bypass and potential unauthorized database access.

**Root cause:** Unsafe query construction combined with incomplete keyword filtering.

**Remediation:**
- Use parameterized queries.
- Eliminate SQL keyword blacklists as the primary defense.
- Restrict database permissions.
- Use consistent authentication checks.
- Return generic authentication errors.

### F-03 — Client-Side-Only Input Validation

**Severity:** Medium

**Affected component:** `script.js`

**Description:** Login validation was performed in browser-side JavaScript. Direct HTTP requests bypassed these checks.

**Impact:** Malicious or unexpected input could reach the server without passing the intended frontend restrictions.

**Root cause:** Validation rules were not enforced consistently on the server.

**Remediation:**
- Implement server-side validation.
- Use client-side checks only for usability.
- Apply prepared statements and contextual output encoding.
- Test endpoints independently of the browser interface.

### F-04 — Insufficiently Protected Admin Login

**Severity:** Medium

**Affected component:** `adminLogin007.php`

**Description:** A separate administrative login endpoint was discoverable through the public page source.

**Impact:** The endpoint provided an additional target for credential attacks and became a path to administrative access when combined with exposed credentials.

**Root cause:** The security of the endpoint depended partly on obscurity.

**Remediation:**
- Enforce server-side role checks.
- Restrict administrative access appropriately.
- Enable MFA.
- Add rate limiting and monitoring.
- Ensure all login paths use consistent security controls.

### F-05 — Leaderboard Tampering

**Severity:** High

**Affected component:** `edit_leaderboard.php`

**Description:** The application accepted manipulated leaderboard values without sufficient authorization and input validation.

**Impact:** An authenticated low-privilege user could alter published leaderboard data.

**Root cause:** Insufficient server-side authorization and validation for state-changing requests.

**Remediation:**
- Verify permissions on every write operation.
- Validate all submitted values.
- Implement CSRF protections.
- Maintain audit logs.
- Restrict editing functionality to authorized roles.

### F-06 — SSTI Leading to Remote Code Execution

**Severity:** Critical

**Affected component:** Profile update functionality and dashboard rendering

**Description:** User-controlled profile data was evaluated as template code. Controlled testing demonstrated command execution as `www-data`.

**Impact:** Server-side command execution and access to resources permitted to the web server account.

**Root cause:** User input was treated as executable template source.

**Remediation:**
- Never evaluate user input as template code.
- Use ordinary template variables.
- Enable output escaping.
- Restrict dangerous functions and callbacks.
- Enforce least privilege.
- Monitor web-server process behavior and outbound connections.

### F-07 — Exposed phpMyAdmin and Directory Information

**Severity:** Medium

**Affected component:** `/phpmyadmin/`, `/flags/`, and application directories

**Description:** The database administration interface was reachable through the public web application, and enumeration revealed several directory paths.

**Impact:** Additional authentication surfaces and useful information were exposed to potential attackers.

**Root cause:** Administrative interfaces and application structure were unnecessarily exposed.

**Remediation:**
- Restrict phpMyAdmin to trusted management networks.
- Remove unused administrative interfaces.
- Disable directory indexing.
- Store sensitive files outside the public web root.
- Keep exposed services and dependencies updated.

### F-08 — Insecure File Permissions and Swap File

**Severity:** Medium

**Affected component:** Web root, `.conn.php.swp`, and `injecticsService.php`

**Description:** The web root contained a leftover editor swap file and application scripts with unsafe write permissions.

**Impact:** Accessible backup files may disclose sensitive information, while writable scripts can create opportunities for unauthorized modification.

**Root cause:** Inadequate deployment hygiene and filesystem permission management.

**Remediation:**
- Remove swap and backup files from deployments.
- Use appropriate file ownership.
- Prevent the web process from modifying application code.
- Move service scripts outside the web root.
- Audit filesystem permissions regularly.

### F-09 — Information Disclosure and Cookie Hardening

**Severity:** Low

**Affected component:** HTML source, HTTP responses, dashboard output, and PHP session cookie

**Description:** Developer comments, server version information, runtime output, and insufficient session cookie hardening exposed unnecessary details.

**Impact:** The disclosures could help attackers understand the environment and increase the impact of other vulnerabilities.

**Root cause:** Excessive diagnostic information and incomplete security configuration.

**Remediation:**
- Remove internal comments and debug output.
- Minimize server banner information.
- Use generic error messages.
- Configure `HttpOnly`, `Secure`, and appropriate `SameSite` cookie attributes.
- Avoid exposing runtime account details to users.

---

## 🏁 Flags Captured

Both challenge flags were successfully recovered.

### Flag 1 — Admin Panel

`THM{INJECTICS_ADMIN_PANEL_007}`

**Discovery method:** Accessed the administrative dashboard using credentials discovered during the assessment.

### Flag 2 — Server Access

`THM{5735172b6c147f4dd649872f73e0fdea}`

**Discovery method:** Retrieved the flag file from the application's flags directory after obtaining the limited web-server shell.

These flags confirm completion of the room objectives documented in the assessment.

---

## 🛡️ Remediation Roadmap

The findings were grouped by urgency to help prioritize corrective actions.

### Immediate priority

1. Remove publicly accessible logs and sensitive files.
2. Rotate exposed credentials and eliminate hardcoded default accounts.
3. Replace unsafe SQL queries with prepared statements.
4. Remove the template injection vulnerability.
5. Restrict dangerous server-side functions and callbacks.

### Short-term priority

1. Enforce authorization on leaderboard modifications.
2. Add server-side validation and CSRF protection.
3. Restrict phpMyAdmin to trusted networks.
4. Protect administrative login functionality with MFA and rate limiting.
5. Correct application file ownership and permissions.
6. Remove swap files and move sensitive files outside the web root.

### Routine security improvements

1. Remove developer comments and unnecessary debug output.
2. Harden session cookie attributes.
3. Reduce unnecessary server version disclosure.
4. Monitor suspicious application behavior.
5. Test authentication and authorization controls during future releases.
6. Include security checks in the deployment pipeline.

---

## 🧠 Key Technical Learnings

This lab strengthened my practical understanding of several important web application security concepts.

### 1. Reconnaissance and enumeration

I learned how port scanning, service identification, directory discovery, and source inspection can reveal the application's attack surface.

### 2. SQL injection

The lab demonstrated why dynamically constructed SQL queries are dangerous and why prepared statements are essential for secure database access.

### 3. Client-side versus server-side validation

I learned that browser-side restrictions can be bypassed and that all security-critical checks must be implemented on the server.

### 4. Authentication and authorization

The engagement highlighted the difference between proving a user's identity and verifying that the user is permitted to perform a specific action.

### 5. Server-Side Template Injection

I explored how incorrectly handled user input can be interpreted as template source and how that behavior can lead to severe server-side vulnerabilities.

### 6. Remote Code Execution

The lab provided practical exposure to validating command execution and understanding the permissions of the process that executes the command.

### 7. Linux filesystem security

The post-exploitation phase highlighted the importance of file ownership, least privilege, deployment hygiene, and protecting sensitive application files.

### 8. Penetration testing documentation

I practiced organizing findings into a technical report with evidence, impact, reproduction details, severity, and actionable remediation recommendations.

---

## 🧩 Challenges and Problem-Solving

One of the important aspects of this lab was understanding why apparently reasonable security measures did not prevent exploitation.

### Challenge 1: Credentials did not work on the standard login

The credentials found in the exposed log were rejected by the standard login page. Further investigation revealed a separate administrative authentication path.

**Learning:** Authentication behavior can differ between endpoints. Each authentication flow must be tested and secured consistently.

### Challenge 2: Frontend validation appeared to block input

The browser-side JavaScript rejected certain input patterns, but the server still accepted directly submitted requests.

**Learning:** Browser controls do not replace server-side validation. HTTP endpoints must be tested independently.

### Challenge 3: Template input was evaluated unexpectedly

A profile field that should have contained ordinary text evaluated a mathematical expression.

**Learning:** User-controlled data must remain data throughout the rendering process. Template evaluation should never be applied to untrusted input.

### Challenge 4: The web shell did not provide root access

The shell operated as `www-data`, not as the root user.

**Learning:** The execution context determines which resources can be accessed. RCE and privilege escalation are distinct security outcomes.

### Challenge 5: Some filesystem risks were not fully validated

The swap file and unsafe permissions were identified, but the swap file contents were not retrieved.

**Learning:** Reports must distinguish confirmed exploitation from potential impact. Findings should reflect the available evidence rather than unsupported assumptions.

---

## 📸 Evidence and Documentation

The full assessment report contains 56 numbered screenshots documenting the engagement, including:

- Network scan results.
- Application pages and source code.
- Directory enumeration output.
- Exposed log file observations.
- Authentication testing.
- Burp Suite request analysis.
- Leaderboard modification.
- Administrative dashboard access.
- Template expression evaluation.
- Command execution confirmation.
- Reverse-shell connection.
- Final filesystem enumeration and flag recovery.

### Detailed penetration testing report

The complete PDF report contains:

- Executive summary.
- Scope and environment.
- Reconnaissance and enumeration.
- Attack chain overview.
- Nine detailed security findings.
- Captured flags.
- Engagement timeline.
- Prioritized remediation roadmap.
- Evidence gallery.

## 📈 Overall Assessment

The Injectics assessment demonstrated that the application could be compromised through a chain of weaknesses involving information disclosure, SQL injection, insufficient authorization, and server-side template injection.

The most critical issue was SSTI leading to remote code execution. Exposed default credentials and SQL injection provided additional routes to unauthorized access, while weaker controls elsewhere increased the overall attack surface.

The findings also demonstrate that fixing a single visible symptom is insufficient when the underlying application design remains insecure. Effective remediation requires secure database access, consistent authentication and authorization, safe template rendering, appropriate filesystem permissions, and careful handling of sensitive information.

The assessment achieved its lab objectives, including recovery of both flags. It also provided practical experience in validating vulnerabilities, distinguishing confirmed findings from potential risks, and translating technical observations into remediation guidance.

## 🏆 Conclusion

Completing the Injectics CTF was a valuable hands-on exercise in web application penetration testing.

Throughout the engagement, I worked through reconnaissance, enumeration, vulnerability discovery, authentication testing, SQL injection, access control analysis, SSTI, controlled command execution, Linux enumeration, and technical reporting.

More importantly, the lab reinforced the importance of understanding the root cause of each vulnerability and the way different weaknesses can combine to create a much greater security risk.

I documented nine findings and included evidence and remediation recommendations to make the work useful as both a learning resource and a reference for secure application development.

I will continue building my practical skills through authorized CTF challenges, vulnerable lab environments, web application security research, and structured penetration testing exercises.

**Lab completed. Both flags captured. Findings documented. Learning continues.** 🚀

---

## ⚠️ Disclaimer

This repository is intended exclusively for educational purposes, authorized penetration testing, and cybersecurity skill development.

All testing described in this write-up was performed against the designated TryHackMe Injectics lab environment.

The techniques documented here must not be used against systems, applications, networks, or accounts without explicit authorization from the owner. The author does not support unauthorized access, data theft, disruption, or malicious activity.

## 👨‍💻 Connect With Me

**Nishant Saini**  
BCA Graduate | Cybersecurity & Ethical Hacking Learner | Aspiring Penetration Tester

- **GitHub:** https://github.com/nishantsaini5786
- **LinkedIn:** https://www.linkedin.com/in/nishant-saini-6a695b322/
- **Portfolio:** https://nishant-official.netlify.app
- **Instagram:** https://www.instagram.com/nishantt_sainii/

---

<p align="center">
  <b>Learn ethically. Test responsibly. Document everything. Keep improving. 🔐</b>
</p>
