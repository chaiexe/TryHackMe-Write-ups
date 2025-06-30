**<p align="center">Hacking Challenge Day 9 - Topic: Security Logging and Monitoring Failures</p>**
---

[![Day 9 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%209%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed: 6/30/2025**

🔒 Topic: Security Logging and Monitoring Failures (OWASP A09 - 2021)

<p align="center">+++++++++</p>

This vulnerability happens when a web application doesn’t properly record or monitor important security events, like failed logins, suspicious activity, or system errors. Without good logging and alerting, attackers can break in, move around, or steal data without being noticed.

**Why it matters:**

 If there's no record of what’s happening or no alerts when something suspicious occurs, security teams won’t know there's a problem until it’s too late.

**Example:**

If someone tries to guess passwords many times and the app doesn’t log those failed attempts or alert anyone, an attacker could brute-force a login unnoticed.

The information stored in logs should include:
- HTTP status codes
- Time Stamps
- Usernames
- API endpoints/page locations
- IP addresses

**Some signs of suspicious activity are:**
- **Too many failed login attempts** or trying to access things like admin pages without permission.
  
- **Requests coming from strange IP addresses or locations**, which could mean someone else is trying to break into an account (though sometimes it's just a false alarm).

- **Very fast or unusual requests**, which might mean someone is using automated tools like bots or scripts.

- **Known attack patterns**, like specific code or commands attackers often use to test for weaknesses in a site.

<p align="center">+++++++++</p>

**<p align="center">📋Security Logging and Monitoring Failures (Challenge)📊</p>**

Following the THM lab instructions, I downloaded the task file. This is the sample file I’ll be examining.

The logs show dated successful and redirected login attempts from multiple users and their IP addresses. 

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/09-Security-Logging-and-Monitoring-Failures/Images/Screenshot%201.png)

The unauthorized redirects are associated with the IP address **49.99.13.16**, while the rest of the login attempts in the logs appear to be successful logins from different IP addresses linked to legitimate users. 

There were four unauthorized redirect attempts from the same IP address, each happening every five minutes. This pattern suggests a **brute force attack** where the attacker gradually increased their privileges from admin to root.

**Lessons Learned:**

This lab reminded me how essential proper security logging and monitoring are for quickly detecting and responding to attacks. Without detailed logs and timely alerts, suspicious activities like repeated unauthorized redirects and privilege escalations can go unnoticed, allowing attackers to take control of critical parts of the system. It showcased how valuable logs are for deeply understanding security incidents. 

<p align="center">+++++++++</p>

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
