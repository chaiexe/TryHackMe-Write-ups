**<p align="center">TryHackMe: Hydra</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Room%20Icon.png" alt="image alt" width="150" />
</p>

📍This write-up is part of my entry into TryHackMe’s Hack2Win Challenge: Room 1, Day 1.

**Topic:** Learning the Hydra Pentesting Tool

**Completed:** 9/1/2025

"Learn about and use Hydra, a fast network logon cracker, to bruteforce and obtain a website's credentials. "

<p align="center">+++++++++</p>

**<p align="center">Task 1: Hydra Introduction</p>**

**What is Hydra?**

**Hydra** is a brute force online password cracking program.

Hydra automates brute-force attacks by rapidly testing a list of passwords against authentication services (like SSH, FTP, web forms, or SNMP) to find the correct one.

Weak or default passwords (like admin:password) are easily guessed from massive wordlists, so always use strong, unique credentials and change defaults immediately.

Finally, Hydra can be installed on Ubuntu with `apt install hydra`, on Fedora with `dnf install hydra`, or downloaded from its official THC-Hydra repository.


**<p align="center">Task 2: Using Hydra (Challenge)</p>**

After identifying the target IP, I ran an Nmap scan, which revealed two open ports:
- 22 (SSH)
- 80 (HTTP)

Accessing the target via a web browser led to a login page.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%201.png)

From the challenge question in Task 2, *“Use Hydra to bruteforce molly’s web password. What is flag 1?”*, we know the username is `molly`. Which means we need to brute-force her password using Hydra to obtain the first flag.

Before jumping into Hydra, I used BurpSuite along with the Firefox Burp proxy to capture an HTTP POST request when submitting test credentials on the login form. This revealed the parameters being used:
```
username=tester&password=password123
```
These POST parameters are needed for constructing the Hydra command, as they define how the login page processes authentication attempts.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%202.png)

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

The Hydra command needed in this task is:
```
hydra -l <username> -P <wordlist> MACHINE_IP http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```
- `-l <username>`: Specifies the single username to attempt (`molly` in this case).
- `-P <wordlist>`: Points to the wordlist file containing potential passwords (e.g., `rockyou.txt`).
- `MACHINE_IP`
- `http-post-form`: Defines the POST parameters: `^USER^` and `^PASS^` are placeholders Hydra replaces with usernames and passwords, and `F=incorrect` tells Hydra the string that indicates a failed login attempt.
- `"/:username=^USER^&password=^PASS^:F=incorrect"`
- `-V`: Enables verbose mode, so Hydra prints each login attempt as it runs.

<p align="center">+++++++++</p>

The error message displayed after an incorrect login attempt is:
```
Your username or password is incorrect.
```

This string is entered into the `F=incorrect` field in the Hydra command to indicate a failed login attempt.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%203.png)

With all required information gathered, the following Hydra command was executed, successfully revealing `molly`’s web password:
```
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.201.118.167 http-post-form "/login:username=^USER^&password=^PASS^:Your username or password is incorrect" -V
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%204.png)

Using the obtained credentials to log in to the web page revealed the first flag of the room!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%205.png)

Next, Hydra was used to brute-force `molly`’s SSH login. The following command was executed, successfully obtaining her SSH password:
```
hydra -l molly -P /usr/share/wordlists/rockyou.txt ssh://10.201.118.167
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%206.png)

I then proceeded to successfully log in to Molly’s SSH account using the password obtained from Hydra. From there, I was able to retrieve the final flag of the room using the `ls` and `cat` commands!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%207.png)

I didn’t get any points from completing this room, but I did gain 2 gold raffle tickets towards the Hack2Win TryHackMe challenge! 

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Hydra/Images/Screenshot%208.png)

**Lessons Learned**

This room taught me another valuable way to use Hydra. I had never used the `http-post-form` flag to brute-force a web login page before, so learning how to construct and execute this command was an awesome learning experience.

Happy learning!

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Hydra Room](https://tryhackme.com/room/hydra?ref=blog.tryhackme.com)
- [THC-Hydra repository](https://github.com/vanhauser-thc/thc-hydra)
