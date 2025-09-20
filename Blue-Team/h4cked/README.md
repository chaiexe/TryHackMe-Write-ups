**<p align="center">TryHackMe: h4cked</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/room%20icon.png" alt="image alt" width="250" />
</p>

**Topic:** Incident response

**Completed:** 9/4/2025

**Objective:** “Find out what happened by analysing a `.pcap` file and hack your way back into the machine.” 

<p align="center">+++++++++</p>

### <p align="center">Task 1: Oh no! We've been hacked!</p>

**Q1: It seems like our machine got hacked by an anonymous threat actor. However, we are lucky to have a `.pcap` file from the attack. Can you determine what happened? Download the `.pcap` file and use Wireshark to view it.**

Following the instructions, I downloaded the `.pcap` file to my VM and opened it in Wireshark to investigate the attack.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%201.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%202.png)

**Q2: The attacker is trying to log into a specific service. What service is this?**

Right-clicking the first stream and using **Follow > TCP Stream** helped me narrow down my search.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%203.png)

Opening the first TCP stream reveals a lot of telling information. The attacker seems to be trying to brute-force their way into an FTP server, targeting the user account `jenny`. There are also a lot of FTP protocol usage reports within the logs themselves. 

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%204.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%205.png)

🇦 ~ `FTP`

**Q3: There is a very popular tool by Van Hauser which can be used to brute force a series of services. What is the name of this tool? **

The TryHackMe hint points to Van Hauser’s GitHub (https://github.com/vanhauser-thc), which reveals the tool name.

🇦 ~ `Hydra`

**Q4: The attacker is trying to log on with a specific username. What is the username?**

From the TCP stream we can clearly see the username being targeted during the brute-force attempts.

🇦 ~ `Jenny`

**Q5: What is the user's password?**

By searching through the TCP streams (using the stream navigation arrows in Wireshark), I found a `230 Login successful` response in stream 7, confirming a successful brute-force and revealing the password used.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%206.png)

🇦 ~ `password123`

**Q6: What is the current FTP working directory after the attacker logged in?**

Stream 16 shows the attacker issuing the `PWD` command on the FTP server, revealing the current working directory.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%207.png)

🇦 ~ `/var/www/html`

**Q7: The attacker uploaded a backdoor. What is the backdoor's filename?**

Scrolling down the same stream shows the attacker ran `STOR` then `chmod 777` to upload a `.php` webshell and give it read/write/execute permissions for owner, group, and others.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%208.png)

🇦 ~ `shell.php`

**Q8: The backdoor can be downloaded from a specific URL, as it is located inside the uploaded file. What is the full URL?**

Stream 18 shows the full contents of `shell.php`, which includes the backdoor source URL.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%209.png)

🇦 ~ `http://pentestmonkey.net/tools/php-reverse-shell`

**Q9: Which command did the attacker manually execute after getting a reverse shell?**

On stream 20 the attacker got a limited shell using Jenny’s account, then ran `whoami` to verify which user they were acting as.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2010.png)

🇦 ~ `whoami`

**Q10: What is the computer's hostname?**

By scrolling the same TCP stream, I followed the attacker’s step-by-step privilege escalation of Jenny’s account; once the shell was stabilized, the system hostname was revealed.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2011.png)

🇦 ~ `wir3`

**Q11: Which command did the attacker execute to spawn a new TTY shell?**

In the previous screenshot, we see the command the attacker used to spawn a new TTY shell; along with the answer to the following question.

🇦 ~ `python3 -c 'import pty; pty.spawn("/bin/bash")'`

**Q12: Which command was executed to gain a root shell?**

🇦 ~ `sudo su`

**Q13: The attacker downloaded something from GitHub. What is the name of the GitHub project?**

After gaining root, the attacker used `git` to pull a repo:

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2012.png)

🇦 ~ `Reptile`

**Q14: The project can be used to install a stealthy backdoor on the system. It can be very hard to detect. What is this type of backdoor called?**

By definition, a stealthy system backdoor that hides itself is a:

🇦 ~ `Rootkit`

### <p align="center">Task 2: Hack your way back into the machine</p>

**Q1: The attacker has changed the user's password! Can you replicate the attacker's steps and read the `flag.txt`? The flag is located in the `/root/Reptile` directory. Remember, you can always look back at the `.pcap` file if necessary. Good luck!**

🇦 ~ `Now for the really fun part, let’s replicate.`

**Q2: Run Hydra (or any similar tool) on the FTP service. The attacker might not have chosen a complex password. You might get lucky if you use a common word list.**

Using the following Hydra command allowed me to crack Jenny’s FTP server password. Since I already knew the targeted username, I didn't need to use a username list.
```
hydra -l jenny -P /usr/share/wordlists/rockyou.txt ftp://10.201.37.187
``` 
![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2013.png)

**Q3: Change the necessary values inside the web shell and upload it to the webserver**

After logging into the FTP server with `jenny`’s credentials, I tried to download `shell.php` but couldn’t, so I pulled the PentestMonkey reverse-shell, updated the target IP/port, saved it as `myturn.php`, and uploaded it.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2014.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2015.png)

I adjusted the required inputs (target IP and port for Netcat) and confirmed the payload was named `myturn.php`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2016.png)

I then logged back into the FTP server as `jenny` and uploaded my payload in the same way the attacker did.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2017.png)

**Q4: Create a listener on the designated port on your attacker machine. Execute the web shell by visiting the .php file on the targeted web server.**

Using the following Netcat command, I set up a listener on port `4546` using Netcat. Navigating to `http://10.201.37.187/myturn.php` in a browser executed the payload and gave me a reverse shell:
```
Nc -lvnp 4546
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2018.png)

I then proceeded to stabilize the shell using the following commands then confirmed my user ID:
```
python3 -c ‘import pty; pty.spawn(“/bin/bash”)’

export TERM=xterm
```
![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2019.png)

**Q5: Become root!**

To confirm which users on the server had `/home` directories:
```
cat /etc/passwd | grep /home
```

I then switched to the `jenny` account (password known) using `su`, verified her `sudo` privileges with `sudo -l` to confirm she could escalate to root, and switched to root using `sudo su`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2020.png)

**Q6: Read the flag.txt file inside the Reptile directory**

Navigating to `/root`, I found a folder named `Reptile` containing the `flag.txt` file, which I viewed using the `cat` command.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/h4cked/Images/Screenshot%2021.png)

🇦 ~ `ebc{REDACTED}d0fd`

### Lessons Learned

I had a lot of fun working through this room, it was a perfect purple team exercise. It was a nice refresher on using Wireshark to analyze a `.pcap` file, follow TCP streams, and understand how an attacker successfully compromised a system to escalate privileges to root. Afterwards, I was able to circle back around and reconstruct how they did it on my own.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**
- [TryHackMe’s h4acked Room](https://tryhackme.com/room/h4cked)
