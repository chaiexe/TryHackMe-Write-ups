**<p align="center">Hacking Challenge Day 7 - Topic: Identification and Authentication Failures</p>**
---
[![Day 7 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%207%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed: 6/25/2025**

🔒 Topic: Identification and Authentication Failures (OWASP A07 - 2021)

🎯 Objective: Continue progressing through the OWASP Top 10 - 2021 room on TryHackMe

<p align="center">+++++++++</p>

**Identification and Authentication Failures** occur when a web application doesn’t properly confirm a user's identity or protect login mechanisms, making it easier for attackers to gain unauthorized access.

**Authentication** is the process that allows users to access web applications by confirming their identity. The most common method involves entering a username and password, which the server checks for validity. If the credentials are correct, the server issues a **session cookie** to the user’s browser. This cookie is essential because HTTP(S) is a stateless protocol. This means each request is independent and doesn’t inherently retain user information. By using a session cookie, the server can maintain the user’s session and accurately associate incoming requests with the correct identity.

**Some common issues with authentication systems include:**

   - **Brute Force Attacks:** Attackers try many username-password combinations until they find the correct one. Without limits on login attempts, this can be highly effective.

   - **Weak Passwords:** Simple passwords like “password1” are easy to guess, making accounts vulnerable. That’s why strong password rules are important.

   - **Weak Session Cookies:** Session cookies help the server remember who a user is. But if the cookie values are easy to guess, an attacker could make their own and pretend to be someone else.

<p align="center">+++++++++</p>

**<p align="center">✨Research Deep-Dive✨</p>**

**Researched Question:** Where are session cookies usually stored within a browser?

**Researched Answer:**

Session cookies are usually stored in the browser’s memory (RAM) and are not written to disk. Because of this, they exist only for the duration of the browsing session, meaning they are deleted when the browser is closed. More specifically:

They are stored in the browser’s cookie storage, accessible through developer tools (`Application` tab in Chrome or `Storage` tab in Firefox).

Unlike persistent cookies, session cookies do not have an expiration date set, so the browser knows to treat them as temporary.

This makes them more secure in some cases, especially on shared or public computers, since they do not persist after the session ends. However, they can still be vulnerable if not handled properly, such as when the `HttpOnly` or `Secure` flags are not set.

<p align="center">+++++++++</p>

**Remediation techniques** depending on the issue:

- To stop password guessing, make sure users are required to create strong passwords.

- To prevent brute force attacks, add a lockout feature that temporarily blocks accounts after too many failed login attempts.

- Use Multi-Factor Authentication (MFA). This adds extra security by requiring something like a code sent to the user’s phone, making it much harder for attackers to break in even if they have the password.

<p align="center">+++++++++</p>

**<p align="center">📍Identification and Authentication Failures (Challenge)📍</p>**

**Objective:** Exploit a logic flaw in the authentication system by bypassing user identity checks through re-registering an existing username with a minor change (like adding a space) to gain unauthorized access to that user's account.

Navigating to the target web application URL `hxxp[:]// 10.10.82.56:8088` opens up to Auth hacks’ authentication web page.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/07-Identification-and-Authentication-Failures/Images/Screenshot%201.png)

**Next Step:** On the registration page, I’ll attempt to register the username `darren` with a leading space `" darren"` to exploit the logic flaw and gain access to an existing account using tester credentials.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/07-Identification-and-Authentication-Failures/Images/Screenshot%202.png)

The message “User registered successfully!” confirmed that the account was created. Logging in with these credentials granted access to Darren’s account, revealing the first flag. This showed that the web application had poor authentication practices, as the newly created account should not have been identified or treated as an existing user.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/07-Identification-and-Authentication-Failures/Images/Screenshot%203.png)

Logging in with the newly registered credentials granted access to Darren’s account, revealing the first flag. The flag represents sensitive data that should not have been exposed to an unauthorized user.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/07-Identification-and-Authentication-Failures/Images/Screenshot%204.png)

Repeating the same process to access Arthur’s account revealed the final flag, confirming the presence of an identification and authentication failure.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/07-Identification-and-Authentication-Failures/Images/Screenshot%205.png)

**Lessons Learned:**

This lab highlighted how simple input validation oversights can lead to serious authentication flaws, allowing adversaries to bypass user identity checks and access sensitive data. It reinforced the importance of sanitizing inputs and properly handling user registration to prevent logic-based vulnerabilities.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
