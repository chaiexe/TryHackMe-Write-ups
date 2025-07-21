**<p align="center">TryHackMe: Madness</p>**
---

<p align="center">
  <img src="x" alt="image alt" width="180" />
</p>

[![Day 19 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2019%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/21/2025

“Will you be consumed by Madness?”

<p align="center">+++++++++</p>

Starting off with an `Nmap -sV` scan revealed two open ports:

- 22 (SSH) OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
- 80 (HTTP) (Apache httpd 2.4.18 ((Ubuntu)))

Navigating to the IP via browser led to the default Apache2 Ubuntu landing page. Viewing the page source revealed the following comment:
```
<!--
    Modified from the Debian original for Ubuntu
    Last updated: 2014-03-19
    See: https://launchpad.net/bugs/1288690
-->
```
This clue hints at a potential vulnerability or misconfiguration, especially since the Apache version in use is 2.4.18, which is known to have several public exploits. A quick search through the provided link led me down a rabbit hole of Apache bugs related to outdated Debian/Ubuntu builds.
