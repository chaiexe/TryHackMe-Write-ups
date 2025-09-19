**<p align="center">Simple CTF</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Simple%20CTF/Images/Icon%20image.png" alt="image alt" width="290" />
</p>

[![Day 15 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2015%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/10/2025

👾 Objective: Root the box.

<p align="center">+++++++++</p>

Beginning with an Nmap scan on the target IP 10.10.23.63 revealed three open ports:

- 21 (FTP)
- 80 (HTTP)
- 2222 (SSH)

Navigating to the webpage prompts the default Apache2 welcome page on Ubuntu, confirming that the server is working correctly after installation. The page should be replaced at `/var/www/html/index.html` before using the server publicly.

![Alt text](1)

Using `gobuster` to brute-force the directories on the web server using the `common.txt` wordlist revealed three accessible paths that returned two successful and one redirect HTTP responses:
- `/index.html`
- `/robots.txt`
- `/simple`

Navigating to `/robots.txt` uncovered a disallowed directory entry `Disallow: /openemr-5_0_1_3`.

![Alt text](2)

Attempting to access `/openemr-5_0_1_3` returned a 404 Not Found error. This could mean the directory has been removed or hidden.

Backtracking to investigate the `/simple` path, redirected to the homepage of CMS Made Simple, a content management system. Noting that the site is powered by CMS Made Simple version 2.2.8.

![Alt text](3)

While researching, an unauthenticated blind time-based SQL injection was discovered on MITRE that exploits the `m1_idlist parameter` (CVE-2019-9053).

This means the web server can be exploited without logging in, by crafting a GET request.

The Python exploit script from Exploit-DB titled “CMS Made Simple < 2.2.10 - SQL Injection” was used to automate the attack. After installing required dependencies   `termcolor` using `pip`, the script was executed successfully,  extracting account credentials along with a salted password hash.

![Alt text](4)

![Alt text](5)

Using Hashcat, the following command was used to uncover the plaintext password for Mitch’s account:
```
hashcat -O -a 0 -m 20 [HASHEDPASSWORD:SALT] /usr/share/wordlists/rockyou.txt
```

![Alt text](6)

![Alt text](7)

Once the password was recovered, it was successfully used to log into the CMS Made Simple admin panel under Mitch’s account, confirming credential validity and granting access to the content management interface.

![Alt text](8)

The recovered credentials were then used to establish an SSH connection as Mitch. After logging in, running `sudo -l` revealed the commands Mitch could execute with elevated privileges.

The output revealed that Mitch could execute`/usr/bin/vim` as root without requiring a password.

![Alt text](9)

![Alt text](10)

Executing Vim with elevated privileges launched the text editor. From there, entering command mode with `:` followed by `!sh` provided a root shell.

Finally, with root access established, retrieving both the `user.txt` and `root.txt` flags was straightforward.

![Alt text](11)

![Alt text](12)

**Lessons Learned**

This challenge improved my post-exploitation skills, particularly in researching and identifying the precise exploit and CVE needed for the target.

It also refreshed my knowledge of using Hashcat for cracking salted hashes, something I hadn’t practiced in a while.

I enjoyed the challenge.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's Easy CTF Room](https://tryhackme.com/room/easyctf)
- [CVE-2019-9053](https://www.cve.org/CVERecord?id=CVE-2019-9053)
- [EDB-ID:46635](https://www.exploit-db.com/exploits/46635)
- [Hashcat Manual](https://hashcat.net/wiki/doku.php?id=hashcat)
