**<p align="center">Simple CTF</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Icon%20image.png" alt="image alt" width="290" />
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

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%201.png)

Using `gobuster` to brute-force the directories on the web server using the `common.txt` wordlist revealed three accessible paths that returned two successful and one redirect HTTP responses:
- `/index.html`
- `/robots.txt`
- `/simple`

Navigating to `/robots.txt` uncovered a disallowed directory entry `Disallow: /openemr-5_0_1_3`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%202.png)

Attempting to access `/openemr-5_0_1_3` returned a 404 Not Found error. This could mean the directory has been removed or hidden.

Backtracking to investigate the `/simple` path, redirected to the homepage of CMS Made Simple, a content management system. Noting that the site is powered by CMS Made Simple version 2.2.8.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%203.png)

While researching, an unauthenticated blind time-based SQL injection was discovered on MITRE that exploits the `m1_idlist parameter`(CVE-2019-9053).

This means the web server can be exploited without logging in, by crafting a GET request.


<p align="center">+++++++++</p>

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%204.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%205.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%206.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%207.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%208.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%209.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%2010.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%2011.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Simple%20CTF/Images/Screenshot%2012.png)

![Alt text](x)

+

Currently being edited ...
