**<p align="center">TryHackMe: RootMe</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/RootMe/Images/RootMe%20Icon.png" alt="image alt" width="100" />
</p>

[![Day 20 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2020%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed:** 7/23/2025

**Topic:** Web application pentesting 

👾 Objective: Root the box.

<p align="center">+++++++++</p>

Starting of with an Nmap scan, I discovered two open ports:

- 22 (SSH) OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
- 80 (HTTP) Apache httpd 2.4.29 ((Ubuntu))

Visiting the IP address in a browser loaded an animated homepage for the HackIT domain, featuring the message: `“Can you root me?”`

![Alt text](1)

Using the Gobuster tool to enumerate the target web server revealed a few interesting directories, `/panel` being the most interesting as it led to a file upload page.

![Alt text](2)

I began testing different file types to see what the upload feature would accept. Standard .php files were rejected, but I was able to successfully bypass the filter by renaming my payload to use a double extension `.php.jpg`

![Alt text](3)

Referring back to my gobuster scan, I was able to confirm the file uploads from the `/panel` directory were stored within the `/uploads` directory.

![Alt text](4)

I also tested a script embedded in a `.phtml` file and discovered that the site was vulnerable to Local File Inclusion (LFI) attacks. This was confirmed by using the URL parameter `?cmd=id`, which returned information about the user and group IDs associated with the current user.

![Alt text](5)

Next, I used the classic Pentestmonkey PHP reverse shell script, updated it with my attack box IP and chosen listener port, and saved it with a `.php.phtml` extension, since the upload page accepted files ending in `.phtml`.

![Alt text](6)

In a second terminal, I set up a Netcat listener on port `5555` while uploading the `.php.phtml` payload via the `/panel` page. 

![Alt text](7)

Navigating to the URL of the tester `.phtml` file activated a callback from the target server to my Netcat listener, granting a reverse interactive shell into the machine.

![Alt text](8)

After some search and discovery (sometimes this process will take way longer than anticipated), I was successful in finding the first `user.txt` flag located in the `var/www` directory.

![Alt text](9)

To identify binaries with SUID (Set User ID) privileges, I ran the following command:
```
find / -perm -4000 -type f 2>/dev/null
```

The binary `/usr/bin/python` looked the most promising, given there are known privilege escalation exploits for certain Python versions.

![Alt text](10)

Using the command `/usr/bin/python -c ‘import os; os.execl(“/bin/sh”, “sh”, “-p”)’`, I was able to exploit the SUID Python binary to gain root access on the server and retrieve the final `root.txt` flag.

![Alt text](11)

**Lessons Learned**

This room taught me that in CTFs, the user flag won’t always be sitting in the user’s home directory, it's important to look beyond the defaults and consider less obvious locations. It also helped reinforce my investigative methodology: start with the smaller findings, then build up from there. I’ve noticed I tend to chase the more challenging paths first and sometimes overlook easier, more direct entry points. But with every room I complete, I’m learning to better balance my curiosity with a structured approach.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's RootMe Room](https://tryhackme.com/room/rrootme)
- [Pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)
- [gtfobins](https://gtfobins.github.io/gtfobins/gdb/)
