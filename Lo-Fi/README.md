**<p align="center">TryHackMe: Lo-Fi</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Lo-Fi/Images/THM%20Lofi%20Icon.png" alt="image alt" width="180" />
</p>

[![Day 17 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2017%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/16/2025

“Want to hear some lo-fi beats, to relax or study to? We've got you covered! “

<p align="center">+++++++++</p>

Beginning with an Nmap scan on the target IP `10.10.187.180` revealed two open ports:

- 22 (SSH)
- 80 (HTTP)

The IP led to a Lo-Fi Music homepage that included multiple YouTube videos.

![Alt text](1)

Viewing the source code, the href `/?page=` parameters looked the most interesting since it indicates that the site if dynamically including content from a `GET`parameter called *page*.

![Alt text](2)

Testing the URL parameters using the `../../../../etc/passwd` successfully revealed sensitive user information confirming local file inclusion.

![Alt text](3)

![Alt text](4)

![Alt text](5)

![Alt text](6)


**Lessons Learned**

x

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Lo-Fi Room](https://tryhackme.com/room/lofi)
