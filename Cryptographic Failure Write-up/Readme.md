**<p align="center">Hacking Challenge Day 2 - Topic: Cryptographic Failure Vulnerabilities</p>**
---
![Day 2 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%202%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)

**Completed: 6/17/2025**

OWASP Top 10 are the top 10 researched and documented web application vulnerabilities currently known.

Today I will be studying the second one - Cryptographic Failure using the TryHackMe platform to study.

<p align="center">+++++++++</p>

**Cryptographic Failures** is the misuse (or lack of use) of cryptographic algorithms for protecting sensitive information.

**Cryptographic failures** occur when developers fail to use strong or proper encryption. This can happen due to improper implementation, misconfiguration, or the improper use of cryptographic algorithms, protocols, or key management practices. The mistake leaves sensitive data (e.g., passwords, credit card details, personal messages, or usernames and passwords) exposed to adversaries for exploitation.

**Examples of What Can Go Wrong:**
- A website saves your password without hiding it - an advesary can see it clearly.
- Your personal info is sent over the internet without proper protection, and an malicious user steals your data.
- A company uses weak encryption that is easy to break.

**Lab Notes:**

Databases are commonly used to store large amounts of data in a format that allows easy access from multiple locations. Most database engines use Structured Query Language (SQL) for querying and managing data.

In production environments, databases are often hosted on dedicated servers running services like **MySQL** or **MariaDB**. However, databases can also exist as single files on disks known as flat-file databases.

**SQLite** is one of the most widely used and lightweight flat-file database formats. It is portable, integrates easily with a variety of programming languages, and includes a built-in command-line interface. **sqlite3** is the command-line tool used to interact with SQLite databases and is pre-installed by default on many Linux distributions.

<p align="center">+++++++++</p>

Following TryHackMe’s sensitive data exposure study material, multiple accounts were discovered while navigating through the SQLite database using the Linux command line. The exposed accounts revealed six user’s names, credit card numbers, and their hashed passwords. The hashes represent real-world examples of weak cryptographic practices.

To demonstrate how easily improperly protected data can be exploited by adversaries, I will attempt to crack three of the discovered hashes provided as part of the TryHackMe ‘OWASP Top 10 - 2021’ room using the online tool [CrackStation](https://crackstation.net/).

**Example 1**
- Hash: 5f4dcc3b5aa765d61d8327deb882cf99
- Result: password

**Example 2**
- Hash: fef08f333cc53594c8097eba1f35726a
- Result: Not found.

**Example 3**
- Hash: b55ab2470f160c331a99b8d8a1946b19
- Result: Not found.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%201.png)

Though not all discovered hashes from the SQLite data were successfully cracked using CrackStation, the fact that one hash was compromised is clear evidence of a cryptographic failure within the database.

<p align="center">+++++++++</p>

**<p align="center">Cryptographic Failures (Challenge)</p>**

**Challenge Objective:** Identify any cryptographic failures present within the targeted database.

According to the THM study material, the web application developer left a note for themselves mentioning there is sensitive data located in a specific directory.

<p align="center">+++++++++</p>

Using Gobuster to enumerate the web application for hidden directories with the command:

```bash
gobuster dir -u hxxp[:]//10.10.152.187:81/ -w /usr/share/dirb/wordlists/common.txt
```
The scan revealed two accessible directories on the web server.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%202.png)
![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%203.png)

Accessing the URL (hxxp[:]//10.10.152.187:81/index.php) displayed the welcome page of Sense and Sensitivity’s homepage, featuring a Login link located at the top right of the page.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%204.png)

Viewing the source code of the (hxxp[:]//10.10.152.187:81/login.php) page uncovered a hidden note the developer left for himself. The note revealed a highly sensitive clue that the web server’s database is stored in the /assets directory

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%205.png)
![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%206.png)

Exploring the (hxxp[:]//10.10.152.187:81/assets) page revealed the web server’s *parent directory, /css, /fonts, /images, /js* directories and its’ web application database named *webapp[.]db*. The database can be downloaded and viewed using the Linux tool **DB Browser for SQlite**

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%207.png)

Using *DB Browser for SQLite* to further investigate the webapp.db database revealed two tables named *sessions* and *users*, indicating the presence of exposed username and password credentials.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%208.png)

Inspecting the users table under the **Browse Data** tab disclosed three user credentials and uncovered four columns named *userID, username, password, and admin*. It was observed that the *admin* column utilizes 1s and 0s to signify which accounts have administrative access, and that each user’s password is hashed.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%209.png)

Using Crackstation to crack the admin user’s MD5 hashed password revealed the plaintext password. 

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%2010.png)

Testing the obtained admin credentials on the web application’s login page (hxxp[:]//10.10.152.187:81/login.php) successfully granted access to the admin account, providing console access and revealing the TryHackMe flag, marking the completion of this write-up.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Cryptographic%20Failure%20Write-up/Images/Screenshot%2011.png)

**Lessons Learned:** The developer leaving a note within the page source that pointed to the /assets directory is an example of **Sensitive Data Exposure**. This insecure practice led to the discovery of the SQLite database file.

Upon examining the database contents, the successful cracking of the admin user's hashed password confirmed the presence of a **Cryptographic Failure** vulnerability. This serves as a strong example of what improper cryptographic security looks like.

<p align="center">+++++++++</p>

🔒*Out of respect for the learning experience, I’ve chosen not to share the final answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.*

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
- [Crackstation](https://crackstation.net/)
