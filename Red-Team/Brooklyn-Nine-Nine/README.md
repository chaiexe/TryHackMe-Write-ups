-----
**<p align="center">TryHackMe: Brooklyn Nine Nine</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Brooklyn-Nine-Nine/Images/Brooklyn%20Nine%20Nine%20Icon.jpeg" alt="image alt" width="150" />
</p>

[![Day 18 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2018%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/18/2025

“This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box. If you find more dm me in discord at Fsociety2006.”

<p align="center">+++++++++</p>

Starting off with an `Nmap -sV` scan revealed three open ports:

- 21 (FTP)
- 22 (SSH)
- 80 (HTTP)

Navigating to the IP address in a browser displayed a background image themed around Brooklyn Nine-Nine.

![Alt text](1)

While investigating the page’s source code, I found an interesting message: 

```
“<!-- Have you ever heard of steganography? →” 
```

The message hints that there may be hidden data within the image.

![Alt text](2)

I downloaded the image exactly as it was served from the web server using:

```
curl -O http://10.10.179.190/brooklyn99.jpg
```

After a bit of research into steganography tools, I decided to try `steghide`, a commonly used tool for embedding and extracting data from images. Using the command:

```
steghide extract -sf brooklyn99.jpg
```

I attempted default passwords like `password` and `admin`. Fortunately, one of them worked, and I extracted a file named `note.txt`.

🚨 **Note:** Steghide doesn’t always require a password to extract embedded content.

![Alt text](3)

Using the newly obtained credentials to successfully SSH into the `holt` user’s computer allowed access to the user’s home directory, revealing the first flag `user.txt`.

![Alt text](4)

After some trial and error exploring ways to escalate privileges, I ran:

```
Sudo -l
```
This revealed that holt could run the following command without a password:

```
(ALL) NOPASSWD: /bin/nano
```
Launching nano with:
```
sudo /bin/nano
```
![Alt text](5)

I then triggered the Execute Command feature using `Ctrl+R`, followed by `Ctrl+X`, which allows for command execution. I typed:

```
cat /root/root.txt
```
And just like that, I retrieved the final root flag, no shell access required!

![Alt text](6)

This was another fun and engaging box! The steganography twist was a cool addition, and the privilege escalation path through nano was clever. Definitely a rewarding experience for Day 18.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Brooklyn Nine Nine Room](https://tryhackme.com/room/brooklynninenine)
