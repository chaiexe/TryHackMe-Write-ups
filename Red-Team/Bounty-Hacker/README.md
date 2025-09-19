**<p align="center">TryHackMe: Bounty Hacker</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/RoomIcon.jpeg" alt="image alt" width="200" />
</p>

[![Day 21 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2021%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed:** 7/28/2025

**Topic:** Web application pentesting 

"You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!"

<p align="center">+++++++++</p>

Starting of with an Nmap scan, I discovered three open ports:

- 21 (FTP) - vsftpd 3.0.3
- 22 (SSH) - penSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
- 80 (HTTP) - Apache httpd 2.4.18 ((Ubuntu))

Navigating to the IP address in a browser loaded a webpage featuring a dialogue between four characters: Spike, Jet, Ed, and Faye, alongside a background image.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%201.png)

Inspecting the page’s source code revealed a reference to a file named `crew.jpg`. I downloaded it using the `curl -O` command.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%202.png)

I tried using Steghide and a few other tools to see if there was any hidden data or metadata in the image, but didn’t uncover anything useful.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%203.png)

I circled back to Nmap to gather a more detailed network scan and learned that the FTP service allowed anonymous login (ftp-anon).

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%204.png)

After successfully logging into the FTP server with the default anonymous credentials, I listed the available files using the `ls` command and found two downloadable `.txt` files. I then used the `get` command to retrieve them.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%205.png)

Viewing the contents with `cat` showed that `locks.txt` contained what looked like a list of passwords, while `task.txt` mentioned a potential username: `lin`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%206.png)

With that information, I used Hydra to brute-force the SSH login credentials, combining the `lin` username with the password list, and it worked. I was able to recover a probable password.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%207.png)

From there, I logged into the server via SSH as lin and successfully captured the first `user.txt` flag for the room!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%208.png)

I followed up with the `sudo -l` command to see what commands Lin could run with root privileges and discovered I’d be able to exploit the `/bin/tar` binary. 

After some research, I learned more about the command needed to exploit this vulnerability:
```
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

This command tells `tar` to:

- Create an archive (`-cf`) to `/dev/null`

- Triggers a checkpoint, which executes `/bin/sh`

- Since it runs with `sudo`, the shell is launched with root privileges

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%209.png)

Entering the command successfully granted root access. From there, I used the `python3 -c 'import pty; pty.spawn("/bin/bash")'` command to stabilize the shell and obtained the final `root.txt` file located within the root directory. - Room complete!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%2010.png)

Out of curiosity, I also tried using Lin’s password with Steghide to access the hidden contents of `crew.jpg`but had no luck, it was worth a try.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bounty-Hacker/Images/Screenshot%2011.png)

**Lessons Learned**

This room taught me that Hydra can also be used to brute-force SSH services, which I hadn’t tried before. It was also a good refresher on basic FTP enumeration and using anonymous login as a potential entry point.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Bounty Hacker Room](https://tryhackme.com/room/cowboyhacker)
- [gtfobins](https://gtfobins.github.io/gtfobins/tar/)
