**<p align="center">Hacking Challenge Day 10 - Topic: Server-Side Request Forgery (SSRF)</p>**
---

[![Day 10 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2010%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed: 7/1/2025**

🔒 Topic: Server-Side Request Forgery (SSRF) (OWASP A10 - 2021)

<p align="center">+++++++++</p>

**Server-Side Request Forgery (SSRF)** is a vulnerability in web applications that allows an attacker to make the server send requests on their behalf, often to internal or protected resources, by exploiting unsanitized user-provided URLs or destinations.

This usually happens because the application lets users provide URLs or destinations for internal processes without proper validation.

**Example of unsanitized user-provided URLs or destinations:**

```
<img src="https://example.com/fetch-image?url=USER_INPUT">
```
A user can provide any URL in `USER_INPUT`, and the server will fetch that URL and display the image.

- Safe input: `https://images.example.com/cat.jpg`

- Malicious input: `http://localhost:8080/admin` or `http://169.254.169.254/latest/meta-data/`

Here, the server blindly trusts the user input and makes the request. An attacker could use it to access internal services or sensitive data.

Instead of the server making safe requests to external services (like APIs), the attacker tricks it into making unauthorized or malicious requests, either:

- Internally: `http://localhost:8080/admin`

- Externally: AWS metadata or internal cloud services
  
**SSRF Can Be Used To:**

- **Discover internal networks** by probing IP addresses and open ports that are not normally exposed to users.

- **Exploit trust relationships between internal services** to access resources or data that are meant to be private or restricted.

- **Communicate with non-HTTP services**, potentially leading to critical attacks like remote code execution (RCE) if the target service is vulnerable.
  
Attackers can use SSRF as a pivot point to move laterally within the internal network, mapping out systems, services, and vulnerabilities that are normally hidden from outside access.

**In short:**
SSRF is when the attacker turns the server into their personal proxy for sending malicious requests. The mindset behind it is "I can’t reach those internal systems myself... but maybe you can, server."

<p align="center">+++++++++</p>

<p align="center">🎲Server-Side Request Forgery (Challenge)🎲</p>

Navigating to the URL: `hxxp[:]//10.10.248.60:8087/` brings up John Woo’s web page. His website showcases his photography portfolio, skills, and a download link to his resume.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/10-Server-Side-Request-Forgery/Images/Screenshot%201.png)

Clicking the “Admin Area” prompts the error message “Admin interface only available from localhost!!!”

Hovering over the “Download Resume” button revels where the server parameter points to:
`secure-file-storage[.]com` within the URL:
`10.10.248.60:8087/downlaod?server=secure-file-storage.com:8087&id=75482342`

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/10-Server-Side-Request-Forgery/Images/Screenshot%202.png)

By using the `nc -l [port]` command to start a listener on the AttackBox, I was able to make the vulnerable application send the request to my machine instead of the secure file storage service. As a result, the API key `THM{Final Flag}` was exposed in the incoming request.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/10-Server-Side-Request-Forgery/Images/Screenshot%203.png)

Going the extra mile to gain access to the admin area.

Using the original vulnerable URL and the knowledge that only localhost are permitted in the admin area.  The `server` parameter is replaced with `localhost:8087/admin%23`to uncover the sensitive data located in the admin area.

 `http://10.10.243.25:8087/download?server=localhost:8087%23&id=75482342`

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/10-Server-Side-Request-Forgery/Images/Screenshot%204.png)

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

*Researched question:* Why is `%23`needed at the end of the `server` parameter?

*Researched Answer:* `%23` is the URL-encoded version of the hash symbol `#`.

In URLs, anything after a `#` is considered a fragment identifier, which is only processed by the browser, not sent to the server. Essentially, the server ignores everything after the `#`, unless you encode it to trick the server into actually reading it.

By using `%23` (the encoded #), you're cutting off anything the application might try to add to the end of your URL. It’s telling the server to ignore everything after the `#`.

The `#` stops the server from processing the rest of the path, which lets you hit the admin page directly, without the app interfering.

<p align="center">+++++++++</p>

**Lessons Learned:** 

This lab was such an eye-opener. I knew SSRF was dangerous, but seeing it in action made it real. The fact that I could redirect a server’s trust and steal something as sensitive as an API key, just by listening with `netcat`, showcased how small misconfigurations can have major impacts. 

What stood out most to me was the power of perspective: instead of attacking directly, SSRF works by whispering to the server, “Do it for me.” That mindset shift helped everything click. It also emphasized how important it is to validate input, restrict outbound traffic, and never assume internal services are safe just because they're "inside."

These types of labs help sharpen my awareness and deepens my respect for how layered web security really is.

Studying the OWASP Top 10 web application vulnerabilities has been really insightful and genuinely fun. I’m looking forward to diving into my next topics.

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/OWASP-Top-10-2021/10-Server-Side-Request-Forgery/Images/e01865ca96de032c241174b728c9d2b1.gif" width="300" alt="Celebration GIF">
</p>

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers
directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
