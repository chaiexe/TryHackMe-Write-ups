**<p align="center">Hacking Challenge Day 2 - Topic: Cryptographic Failure Vulnerabilities</p>**
---
![Day 2 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%203%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)

**Completed: 6/18/2025**

OWASP Top 10 are the top 10 researched and documented web application vulnerabilities currently known.

Today I studied the third one - Command Injection using the TryHackMe platform to study.

<p align="center">+++++++++</p>

**Injections** happen when an adversary is able to submit malicious input into a web application, causing the application to execute unintended commands. The result often leads to unauthorized access, data exposure, or full system compromise.

**Examples:**
- **SQL Injections:** These occur when a web application allows user-controlled input to be included in a database query without proper validation. An adversary can inject malicious SQL code to view, modify, or delete data they shouldn’t have access to (e.g., account credentials or sensitive personal information).

- **Command Injections:** These happen when user input is interpreted as system commands without appropriate validation. An adversary can exploit this vulnerability to run arbitrary commands on the server, potentially gaining access to sensitive data or full system control.

**Remediations:**

- **Implementing whitelists** to only permit trusted and expected input.
- **Sanitize input** by stripping or escaping potentially dangerous characters before processing.

**Note:** *Escaping* means marking special characters in user input so they are treated as plain text, rather than being interpreted as part of a command or code.

**Example of Exploiting Command Injections**

An adversary who exploits a Bash feature called **inline commands**, which lets one run a command inside another command using the format *$(command)*. In the **cowsay** app, this can be abused by injecting input like `$(whoami)`. Bash will execute the inner command first, then pass its result into the outer command. The trick allows adversaries to run arbitrary commands on the server by hiding them inside user input. The **cowsay** cow would say whatever the command returns. 🐮💬

<p align="center">+++++++++</p>

**<p align="center">Command Injection (Challenge)</p>**

Navigating to the website ( hxxp[:]//10.10.242.121:82/ ) presents the Cowsay Online homepage. The page features a form allowing users to input a message and select a cow style, which is then rendered using the **cowsay** command.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%201.png)

Choosing *dragon* as the cow type and *hi* as the message input resulted in an ASCII image of a dragon echoing the word “hi.” This interaction also updated the URL to:

hxxp[:]//10.10.242.121:82/?cow=dragon&mooing=hi

These URL parameters are user-controlled and can be tested for potential command injection vulnerabilities.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%202.png)

Testing the Bash inline command `$(whoami)` within the input field resulted in the ASCII image displaying "apache". This confirms that the web application is vulnerable to command injection and may be susceptible to further exploitation.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%203.png)

A simple inline command, `$(ls)`, was injected into the input field. The server responded by displaying the following files, confirming command execution:
```BASH 
css drpepper.txt index.php js
```
![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%204.png)

Further testing the command injection vulnerability with the cat command was used to read the contents of the drpepper.txt file. The response revealed the message:

*"I'm sorry Mario, but the Dr. Pepper you \ are looking for is in another webapp."*

This indicates that the file is likely a decoy, suggesting the real target may be located elsewhere within the application or on a related service.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%205.png)

Using the `pwd` command to verify the current working directory on the Apache server revealed that the application is operating within the `/htdocs` directory.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%206.png)

**Note:** Command injections cannot run `cd ..` by itself:

- Each injected command usually runs in its own subshell, which doesn’t keep the new working directory.


- So `cd ..` will work temporarily in that moment, but won’t persist between commands.

Using ` $(cd /; ls)` to enumerate the root directory confirmed that the command injection vulnerability allows traversal beyond the web root. The server returned the following contents:

```BASH
bin  cows  dev  docker-entrypoint.sh  etc  
home  htdocs  lib  media  mnt  opt  
proc  root  run  sbin  srv  sys  
tmp  usr  var
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%207.png)

Testing the` $(cd ..; cat /etc/passwd)` command revealed a list of active users along with other valuable information, including usernames, user IDs (UID), group IDs (GID), user descriptions, home directories, and default shells. It also indicated that the actual password hashes are stored securely in the `/etc/shadow` file.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%208.png)

<p align="center">+++++++++</p>

**<p align="center">✨ Research Deep-Dive ✨</p>**

After some research, I learned that the command `$(awk -F: '$3 >= 1000' /etc/passwd)` can be used to identify standard users on the system. It lists all user accounts with a UID of 1000 or higher, which typically represent real human users, and excludes system, service, or daemon accounts such as *root*, *daemon*, or *mysql*.

Additionally, `/sbin` stands for “system binaries”. It’s a directory that contains special programs used by system administrators to manage the system. Standard user’s should not be able to access the `/sbin` directory. 

One of these programs is `/sbin/nologin`, which is often assigned as the login shell for service accounts (such as *apache* or *mysql*). When `/sbin/nologin` appears as a user’s shell in `/etc/passwd`, it indicates that the account is not intended for interactive login and is typically used to run background services or system processes.

<p align="center">+++++++++</p>

These findings confirm that there are no standard users present on the server, making the answer 0.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%209.png)

Viewing the end of the `/etc/passwd` file confirms that the *apache* user’s default shell is set to `/sbin/nologin`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%2010.png)


<p align="center">+++++++++</p>

**<p align="center">✨ Research Deep-Dive ✨</p>**

On Linux/Unix systems, a user's default shell is the program that runs when they log into the system through a terminal or SSH.
**For example**, if a user’s shell is set to:


- `/bin/bash` ~ they get a normal interactive terminal.


- `/bin/sh` ~ they get a more minimal shell.


- `/sbin/nologin` or `/bin/false` ~ they’re not allowed to log in at all. The system ends the session right away.

Though the *apache* user’s default shell may be set to `/sbin/nologin`, meaning it can’t log in, open a terminal, or act independently; it still operates like a robot built to perform specific background tasks - such as serving web pages and executing scripts it's programmed to follow. Command injection works by quietly slipping malicious instructions into that robot’s routine, making it believe the injected commands are just part of its original programming.

<p align="center">+++++++++</p>

Using the `$(cd /; uname -a)`command resulted in detailed kernel information, including its version. 

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%2011.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/3.%20Command%20Injection%20Write-up/Images/Screenshot%2012.png)

Finally, command `$(cd /; cat /etc/os-release)`displayed the Linux distribution and version information in a readable format.

<p align="center">+++++++++</p>

**Lessons Learned:**

This lab deepened my understanding of how critical proper input validation, sanitization, and escaping are in protecting web applications from command injection vulnerabilities. I learned how even trusted services, like those running under restricted users such as apache, can be exploited when user-controlled input is directly passed into system-level commands. The lab highlighted how command injection works by manipulating the logic of server-side scripts - making malicious input appear as though it were part of the original program. It also reinforced the importance of treating all user input as untrusted and implementing secure coding practices to prevent attackers from executing arbitrary commands on the server.

<p align="center">+++++++++</p>

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
