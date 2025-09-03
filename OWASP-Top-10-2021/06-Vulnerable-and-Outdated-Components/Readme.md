**<p align="center">Hacking Challenge Day 6 - Topic: Vulnerable and Outdated Components</p>**
---
[![Day 6 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%206%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed: 6/24/2025**

🔒 Topic: Vulnerable and Outdated Components (OWASP A06 - 2021)

🎯 Objective: Continue progressing through the OWASP Top 10 - 2021 room on TryHackMe

<p align="center">+++++++++</p>

**Vulnerable and Outdated Components** are weaknesses that occur when web applications depend on old or unpatched software, leaving them exposed to known security exploits.

**Example:** A company running an outdated version of WordPress (version 4.6) is exposed to known vulnerabilities like unauthenticated Remote Code Execution (RCE). Tools like WPScan can identify such versions, and public exploit databases (like Exploit-DB) often contain ready-to-use exploits.

**Remediation:** Regularly update software and third-party components, apply patches, and monitor for new vulnerability disclosures to prevent attackers from exploiting known vulnerabilities.

<p align="center">+</p>

**<p align="center">📍Vulnerable and Outdated Components (Challenge)📍</p>**

**Objective:** Confirm the contents of the existing `/opt/flag.txt` file on the vulnerable web application.

Visiting the target page `hxxp[:]//10.10.186.176:84` brings up the CSE Bookstore homepage, which displays the following message:

*“This site has been made using PHP with MySQL (procedure functions)!
The layout use Bootstrap to make it more responsive. It's just a simple web!”*

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%201.png)

There’s a plethora of information to gather just by exploring the site.

One interesting detail is a link at the bottom-left of the page titled projectworlds, which leads to ` hxxps[:]//projectworlds[.]in/`. The site lists free and paid PHP and Java project source codes. This indicates that the vulnerable application may have been built using one of these publicly available templates.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%202.png)

Viewing the page source `view-source:hxxp[:]//10.10.186.176:84/` reveals several potentially outdated components, such as older Bootstrap files and possibly insecure PHP functions, hinting at underlying vulnerabilities.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%203.png)

To investigate further, I googled for "CSE Bookstore exploits", which led to an Exploit-DB entry titled “Online Book Store 1.0 - Unauthenticated Remote Code Execution”

This matched the structure and description of the vulnerable app, confirming that the application in use is publicly known to be exploitable.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%204.png)

The exploit confirmed being related to the projectworlds php homepage as well.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%205.png)

<p align="center">+++++++++</p>

**<p align="center">✨Research Deep-Dive✨</p>**

The “Online Book Store 1.0 - Unauthenticated Remote Code Execution” exploit code targets a specific file upload vulnerability in the `/admin_add.php` endpoint. It abuses this by uploading a malicious PHP web shell disguised as an image file, which is then placed in the `/bootstrap/img/` directory. The structure is also present in the CSE Bookstore site. This links the vulnerable target to the same source code used in the Projectworlds "Online Book Store" project.

**The script does three things:**

1) Uploads the PHP web shell using the `admin_add.php` page.

2) Checks if the upload worked by sending a test command.

3) If it worked, it gives you a remote shell where you can type commands like `whoami` to interact with the server.

<p align="center">+++++++++</p>

Next, I saved the RCE exploit as a `.py` file and made it executable using the `chmod +x` command, which adds execute permissions to the file. This allows me to run the Python script directly from the local terminal.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%206.png)

Running the exploit using `python3` successfully opened a reverse interactive shell in my terminal, giving me command-line access to the target server. I used the `ls` command to list the files on the server and begin exploring the system.

**Note:** A reverse shell allows the target machine to connect back to my terminal, letting me run commands on it remotely as if I were sitting at their keyboard.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%207.png)

Running the `whoami` and `pwd` commands confirmed that I had access as the web server user and that I was inside the `/htdocs/bootstrap/img` directory of the Apache server.

Finally, using the cat command to read the contents of `/opt/flag.txt` revealed the hidden flag, successfully completing the lab.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/06-Vulnerable-and-Outdated-Components/Images/Screenshot%208.png)

**Lessons Learned:**

Completing this lab highlighted the critical risks of using outdated software and publicly available project templates. It showed how attackers can easily find and use known exploits, such as unauthenticated remote code execution, to gain control of a system with minimal effort. By simply exploring the web application, identifying reused components, and connecting them to open-source code, I was able to exploit the vulnerability and access sensitive files. This reinforced the importance of regular updates, careful selection of third-party code, and thorough enumeration when assessing a target’s security defense.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
