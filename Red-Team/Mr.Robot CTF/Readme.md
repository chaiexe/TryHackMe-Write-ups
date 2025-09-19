**<p align="center">Mr Robot CTF</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Mr.Robot%20CTF/Images/1.png" alt="image alt" width="290" />
</p>

[![Day 13 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2013%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/7/2025

"Based on the Mr. Robot show, can you root this box?...There are 3 hidden keys located on the machine, can you find them?" - THM
<p align="center">+++++++++</p>

Beginning with an Nmap scan on the target IP `10.10.29.14` revealed three open ports:
- 22 (SSH)
- 80 (HTTP)
- 443 (HTTPS)

![Alt text](1)

Navigating to the IP address in a browser (`http://10.10.29.14`) led to a themed Mr.Robot terminal interface mimicking #fsociety. The terminal is lightly interactive, displaying a list of commands that provide a basic overview of the website.

![Alt text](2)

![Alt text](3)

Using `gobuster` for directory enumeration uncovered several interesting paths. One that stood out was the `robots.txt` file, usually a good place to find hidden or sensitive directories.

![Alt text](4)

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

**Researched Fact:* A `robots.txt`file is a plain text file placed at the root of a website that tells web crawlers (like Googlebot or Bingbot) what they’re allowed or not allowed to access on the site.

Websites use it to control how search engines index their content. For example, if there are private folders, admin panels, or duplicate pages that a site owner doesn’t want on Google, they can “disallow” them in `robots.txt`.

**Note:** The file is case-sensitive, and typically in lowercase format

<p align="center">+++++++++</p>

The `robots.txt` file revealed two key things: a downloadable dictionary file and the room’s first key.

![Alt text](5)

The `.dic` file was downloaded using the `wget` command. After that, the `curl` command was used to retrieve and view `key-1-of-3.txt`, successfully revealing the first key.

![Alt text](6)

Next, `gobuster` was run again, this time using the newly downloaded dictionary file as a wordlist. This revealed several additional directories that weren’t discovered in the initial scan.

![Alt text](7)

Reviewing each discovered directory, `/license` was the most intriguing as it included a Base64 encoded text located at the bottom of the page.

![Alt text](8)

Using an online Base64 decoder, the encoded text was successfully converted into plain text, revealing a potential username and password combination.

![Alt text](9)

Navigating to `http://10.10.29.14/login` redirected to the WordPress login page at `http://10.10.29.14/wp-login.php`. Logging in with the previously discovered credentials successfully granted access to the WordPress admin account for user Elliot.

![Alt text](10)

Investigating the admin’s account settings uncovered the user’s linked personal website. The website represents sensitive data an adversary could use to gather more information about a target or social engineer.

![Alt text](11)

A PHP reverse shell from Pentestmonkey was uploaded by embedding it into one of the theme’s template files through the Appearance > Editor > Templates section in WordPress.

![Alt text](12)

Accessing the modified `404.php` file triggered the reverse shell, which successfully connected back to the configured Netcat listener, providing a basic interactive shell.

![Alt text](13)

Privilege escalation was achieved by exploiting the vulnerable SUID binary located at `/usr/local/bin/nmap`. Using the interactive mode in this older version of Nmap allowed shell execution as root.

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

The Nmap SUID Exploit: Takes advantage of an older version of Nmap installed with root privileges (via the SUID bit), allowing attackers to access its interactive mode and execute shell commands as root, effectively escalating privileges.

During post-exploitation, I ran the following command to search for SUID binaries:
```
find / -perm -4000 2>/dev/null

```
One of the results was:
```
/usr/local/bin/nmap

```
Since it's an uncommon location and owned by root, I checked if it had the older interactive mode feature:
```
/usr/local/bin/nmap --version

```
Instead of printing the version, it opened an interactive Nmap shell. From there, I used the following command:
```
!sh

```
This dropped me into a root shell. It worked because the Nmap binary had the SUID bit set, meaning it ran with the permissions of its owner (root), so even though the shell was launched by the `daemon` user, it executed with root privileges.
This is a known vulnerability in older Nmap versions like version < 5.21 that allowed command execution through interactive mode.

<p align="center">+++++++++</p>

![Alt text](14)

Successfully completed the mission by gaining access to the final two keys, fully compromising the target machine!!

![Alt text](15)

**Lessons Learned**

This challenge showed how powerful enumeration is, especially when it comes to overlooked files like `robots.txt`. It also showed how user misconfigurations in platforms like WordPress can lead to a full compromise when paired with something as simple as a php reverse shell.

Privilege escalation through the Nmap SUID binary was the real highlight. It showcased how important it is to check for old versions and misused permissions. Even something that looks harmless at first glance can open the door to root if left unchecked.

All in all, this lab was really fun!.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's Mr Robot CTF Room](https://tryhackme.com/room/mrrobot)
