**<p align="center">TryHackMe: Lo-Fi</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/THM%20Lofi%20Icon.png" alt="image alt" width="180" />
</p>

[![Day 17 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2017%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/16/2025

“Want to hear some lo-fi beats, to relax or study to? We've got you covered! “

<p align="center">+++++++++</p>

Beginning with an Nmap scan on the target IP `10.10.187.180` revealed two open ports:

- 22 (SSH)
- 80 (HTTP)

Navigating to the IP in a browser brought up a Lo-Fi Music homepage with multiple embedded YouTube links.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/Screenshot%201.png)

Viewing the source code, the hrefs with `/?page=` parameters looked the most interesting since they indicate the site is dynamically including content based on a `GET` parameter called *page*.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/Screenshot%202.png)

Testing the URL parameter with directory traversal `../` had promising results, indicating the webserver might be vulnerable to Local File Inclusion.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/Screenshot%203.png)

Next, the `/?page=` parameter was tested with `../../../../etc/passwd`, which revealed multiple user accounts on the server, including root.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/Screenshot%204.png)

Unfortunately, navigating to the `../../../../root/root.txt`path did not give a flag.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/Screenshot%205.png)

However, viewing the `flag.txt` file did! 

![Alt text]([6](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Lo-Fi/Images/Screenshot%206.png))


**Lessons Learned**

This room was a great refresher on how easily a web server’s directory can be traversed using URL parameters if proper user input validation is missing. I haven’t used this method in a while so it was fun to revisit.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Lo-Fi Room](https://tryhackme.com/room/lofi)
