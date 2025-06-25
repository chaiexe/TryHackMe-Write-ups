**<p align="center">Hacking Challenge Day 5 - Topic: Security Misconfiguration</p>**
---
[![Day 5 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%205%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed: 6/23/2025**

🔒 Topic: Security Misconfiguration (OWASP A05 - 2021)

🎯 Objective: Continue progressing through the OWASP Top 10 - 2021 room on TryHackMe

<p align="center">+++++++++</p>

**Security misconfiguration** refers to improper implementation or management of security controls, often due to default settings, incomplete setups, or overlooked hardening steps. This class of vulnerability emerges when security mechanisms could be configured correctly, but aren’t.

**Examples:**

**📌 Default Login Credentials Left Unchanged:** 

An admin panel still uses the default username and password like `admin:admin` or `root:toor`, making it easy for attackers to gain access.

**📌 Directory Listing Enabled:** 

When a website allows visitors to see a full list of files in a folder (like `/uploads/`), an attacker can browse and download sensitive files that shouldn't be exposed.

**📌 Overly Verbose Error Messages:** 

A login form that displays an error like “SQL syntax error in line 1 near 'SELECT * FROM users WHERE username='admin'…” gives attackers clues about the backend system, which they can exploit.

<p align="center">+</p>

**Debugging Features:**

One common type of security misconfiguration is leaving debugging features enabled in a production environment. While these tools are helpful during development, they can expose sensitive internal functionality if not properly turned off before deployment. A well-known example of this was the 2015 Patreon breach, where attackers reportedly took advantage of an exposed Werkzeug debug console. This tool, used in Python web applications, allows for arbitrary code execution if accessible. It’s a reminder that even small oversights in configuration can lead to serious security risks.

<p align="center">+++++++++</p>

**<p align="center">🔎 Practical Lab 🧪</p>**

Moving into the practical lab to explore how such misconfigurations can be identified and exploited in real-world scenarios.

Starting the THM target VM, the lab objective is to navigate to `hxxp[:]//10.10.145.40:86` and try to exploit the security misconfiguration to read the application's source code.

<p align="center">+</p>

Navigating to the web page `hxxp[:]//10.10.145.40:86/console` displays an interactive Werkzeug console. The web page reads “In this console you can execute Python expressions in the context of the application. The initial namespace was created by the debugger automatically.”

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/05-Security-Misconfiguration/Images/Screenshot%201.png)

Following THM’s lab instructions to use the Werkzeug console to execute the `ls -l`command using the following Python code:
```
import os; print(os.popen("ls -l").read())
```
**Note:** This command lists the files and directories in the current directory in long format.
`Todo.db` is the database file name in the current directory.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/05-Security-Misconfiguration/Images/Screenshot%202.png)

Modifying the python code to read the contexts of the `app.py` file reveals the `secret_flag` variable in the source code.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/05-Security-Misconfiguration/Images/Screenshot%203.png)

**Lessons Learned:** 

This lab highlighted a real-world scenario of how easily Security Misconfiguration vulnerabilities can be exploited through something as simple as leaving a developer debugging tool enabled. It also emphasized the importance of being mindful during the development phase of a web application. This experience reminded me that in cybersecurity, no detail is too minor to review.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
