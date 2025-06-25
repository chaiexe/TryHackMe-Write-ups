**<p align="center">Hacking Challenge Day 1 - Topic: Broken Access Control / Insecure Direct Object Reference Vulnerabilities</p>**
---
![Day 1 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%201%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)

**Completed: 6/16/2025**

OWASP Top 10 are the top 10 researched and documented web application vulnerabilities currently known.

Today I will be studying the first one - Broken Access Control using the TryHackMe platform to study.

<p align="center">+++++++++</p>

**Broken Access Control** is when a web application fails to restrict a user from what they are allowed and not allowed to access.

- Core Concept: It occurs when users can bypass authorization and act outside their intended permissions.

Examples:
- A regular user being able to access admin functions.

- A user being able to modify account IDs in URLs to access other users data (Insecure Direct Object Reference - IDOR).

- A user being able to view sensitive information from other user’s accounts.

<p align="center">+++++++++</p>

<p align="center">Broken Access Control (IDOR Challenge):</p>

**Insecure Direct Object Reference (IDOR)** is when a user is able to access content on the website they wouldn't normally be able to see as a regular user. This occurs when the website’s programmer improperly places proper access controls around a Direct Object Reference. The IDOR gives access to the requested object without verifying if the user is actually allowed to access the object.

**A Direct Object Reference** is when an **identifier** (like a number or ID) is used to directly access a specific resource on the website’s server — such as a file, user profile, or message. These identifiers are often simple and guessable, which makes them vulnerable if access controls aren’t properly placed.

**Challenge Objective:** Log into the targeted website and see if there is a broken access control vulnerability within the web application.

**Given username:** noot / **Given password:** test1234

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%201.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%202.png)

After successfully logging in, I observed that the URL contained an id parameter: `hxxp[:]//10.10.0.36/note.php?note_id=1`.

The page displays a list of things I, the authenticated user, need to buy from the store. 

To test for potential Insecure Direct Object References (IDORs), I modified the `id` parameter value to `2` to determine if access to another list or additional data would be returned.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%203.png)

The page `hxxp[:]//10.10.0.36/note.php?note_id=2` returned  a list of scheduled events planned for the month of September 2022. At this point, it is unclear whether the data is associated with the currently authenticated user or another account. Further testing of the URLs `id` parameter will be conducted to determine if a potential Broken Access Control or IDOR vulnerability is present.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%204.png)

Testing the `id` parameter with the value `3`, the URL `hxxp[:]//10.10.0.36/note.php?note_id=3` returned the message  “You are not supposed to see this note... Which means you are on the right track!” This message indicated that further unauthorized access is possible.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%205.png)

Testing the `id` parameter with the value `5`, the URL `hxxp[:]//10.10.0.36/note.php?note_id=5` returned a message `“Hint: Do note_ids start from 1? Maybe go lower ;)”` The message suggests that additional context may exist at lower `id`  values and encourages broader parameter testing.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%206.png)

Testing the `id` parameter with the value of `-1` the URL `hxxp[:]//10.10.0.36/note.php?note_id=-1` returned an error page. The `id` value `0` will be tested next.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Broken%20Access%20Control%20Write-Up/Images/Screenshot%207.png)

Finally, testing the URL `hxxp[:]//10.10.0.36/note.php?note_id=0` with the `id` parameter value `0` returned a THM flag confirming the presence of an Insecure Direct Object Reference (IDOR) vulnerability. In this case, the unauthorized resource was a flag and the vulnerability was located in the URL id parameter with the value “0”.

In a real-world scenario, this flag would represent sensitive data an authenticated user was not authorized to access.

**Lessons Learned:**
This TryHackMe lab taught me to broaden my perspective when searching for Broken Access Control vulnerabilities or IDORs. These vulnerabilities exist when developers improperly code a website and leave Direct Object References unprotected. This oversight allows adversaries to exploit the vulnerability and gain unauthorized access to sensitive data they were never intended to view.

<p align="center">+++++++++</p>

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
