**<p align="center">TryHackMe: Source</p>**
---

[![Day 14 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2014%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/8/2025

🎯**Objective:** “Exploit a recent vulnerability and hack Webmin, a web-based system configuration tool.” - THM

<p align="center">+++++++++</p>

Beginning with an Nmap scan on the target IP 10.10.34.213 revealed two open ports:
- 22 (SSH)
- 10000 (HTP)

Navigating to the webpage prompts an error message “Unable to connect”, hinting there must be another way to connect to the server.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%201.png)

According to Tenable, there is a reported Remote Code Execution (RCE) vulnerability for Webmin versions 1.882 to 1.921. The target 10.10.34.213 is running Webmin version 1.890.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%202.png)

Using Metasploit to search for the known CVE:2019-15107 led me to the `linux/http/webmin_backdoor` module.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%203.png)

To properly configure the exploit, I set:
- `SSL` to `true` (since Webmin runs over HTTPS),
- `ForceExploit` to `true` to bypass Metasploit’s internal check, and  
- I selected the payload `cmd/unix/reverse_netcat` for personal preference and easier shell interaction.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%204.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%205.png)

After modifying the options, I successfully ran the exploit and received an unstable Netcat shell.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%206.png)

Next, the golden commands `export TERM-xterm` & `python -c 'import pty; pty.spawn("/bin/bash")' were used to successfully stabilize the shell as `root`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%207.png)

The command `cat /etc/passwd/ | grep /home` was used to locate any user accounts with a `/home` directory. Out of the two that were found, the user `Dark` looked the most intriguing.

Using the command `su - [SUID]` to switch to the `Dark` user’s account allowed me to locate the `user.txt` file located in the user’s home directory.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%208.png)

Since I initially was root, I restarted the exploit session to get back to `root` user and located the final flag using the `/root/root.txt` command.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Source/Images/Screenshot%209.png)

Mission accomplished!!

<p align="center">+++++++++</p>

<p align="center">🕹️Notable Commands Learned🕹️</p>

1) This command helps stabilize the shell’s terminal behavior. It removes and glitches with the display, colors, and input handling, making the user experience smoother to use.
```
export TERM=xterm
```
2) This command allows for a nicer interactive shell if Python is installed on the system.
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
3) This command helps to see what user you currently are.
```
id
```
4) This command lists all executable files on the system that run with elevated privileges (SUID) 
```
find / -perm -4000 -type f 2>/dev/null
```
5) Lists normal user accounts who have a home directory under `/home`, which is where you’ll often find user flags, SSH keys, bash history.
```
cat /etc/passwd | grep /home
```
6) Command switches the user to whatever SUID chosen
```
su - [SUID]
```
<p align="center">+++++++++</p>

**Lessons Learned**

This lab was a refresher of using Metasploit. I got more comfortable with setting exploits, configuring payloads, and understanding how Metasploit handles listeners on its own. I ran into a shell that gave no prompt, which reminded me how useful `python -c 'import pty; pty.spawn("/bin/bash")'` and `export TERM=xterm` are for stabilizing access. I also revisited SUID enumeration with `find / -perm -4000 -type f 2>/dev/null` and reviewed how `pkexec` can be a potential vector for privilege escalation via the PwnKit vulnerability. But in the end, I remembered I already had a root shell from an earlier session, and used it to grab the root flag. 😄

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's Source Room](https://tryhackme.com/room/source)
- [Tenable’s CVE-2019-15107 Article On The Webmin Vuln](https://www.tenable.com/blog/cve-2019-15107-exploit-modules-available-for-remote-code-execution-vulnerability-in-webmin)
- [Offsec’s Msfconsole Commands Listing](https://www.offsec.com/metasploit-unleashed/msfconsole-commands/#msfconsole-commands)
