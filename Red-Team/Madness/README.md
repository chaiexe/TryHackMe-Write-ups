**<p align="center">TryHackMe: Madness</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Time%20loop.gif" alt="image alt" width="300" />
</p>

[![Day 19 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2019%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/21/2025

“Will you be consumed by Madness?”

<p align="center">+++++++++</p>

I began with an `Nmap -sV` scan revealed two open ports:

- 22 (SSH) OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
- 80 (HTTP) (Apache httpd 2.4.18 ((Ubuntu)))

Navigating to the IP via browser led to the default Apache2 Ubuntu landing page. Viewing the page source revealed the following comment:
```
<!--
    Modified from the Debian original for Ubuntu
    Last updated: 2014-03-19
    See: https://launchpad.net/bugs/1288690
-->
```
This clue hints at a potential vulnerability or misconfiguration, especially since the Apache version in use is 2.4.18, which is known to have several public exploits. A quick search through the provided link led me down a rabbit hole of Apache bugs related to outdated Debian/Ubuntu builds.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%201.png)

After some initial dead ends, I took a closer look at the HTML source code and found something more promising:
```
 <div class="page_header floating_element">
        <img src="thm.jpg" class="floating_element"/>
<!-- They will never find me-->
        <span class="floating_element">
          Apache2 Ubuntu Default Page
        </span>
      </div>
```
The hidden comment, “They will never find me”, combined with the presence of thm.jpg, stood out as a clue worth digging into.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%202.png)

The `curl -O` command was used to download the image file. To investigate whether it contained any hidden data, I attempted to extract contents using Steghide. However, I was unsuccessful, as none of the default or common passwords worked.

To verify the nature of the file, I ran:
```
file thm.jpg
```

The output returned as `data`, suggesting the file might be corrupted or incorrectly labeled, not a valid `.jpg` or `.png` despite its extension.
To dive deeper, I used the command:

```
xxd thm.jpg | head
```

This command displays the file’s hexadecimal signature, which can help identify the actual file type.
From my research, I learned that:

- A valid JPEG file starts with the hex value `FFD8`
- A valid PNG file starts with `89 50 4E 47`

These values can be critical indicators when checking file authenticity or uncovering hidden data.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%203.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%204.png)

This is where things started to get interesting. The image file's header contained the PNG signature bytes, but its tail ended with JPEG hex values. This mismatch suggested that the file was mislabeled or tampered with, and needed to be corrected to a proper JPEG format before it could be analyzed further.

To fix the header, I used the following command:
```
printf '\xff\xd8\xff\xe0\x00\x10\x4a\x46\x49\x46\x00\x01' | dd conv=notrunc of=thm.jpg bs=1
```

**Command Breakdown**
- `printf '\xff\xd8\xff\xe0\x00\x10\x4a\x46\x49\x46\x00\x01'`: This prints raw hex bytes that make up the standard JPEG header.

- `dd`: A Unix utility for low-level file operations. In this case, it's used to write the corrected header into the file.

- `conv=notrunc`: Prevents the file from being truncated, only the beginning is overwritten.

- `bs=1`: Sets the block size to 1 byte to ensure precise byte-by-byte writing.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%205.png)

After replacing the incorrect header, the file thumbnail updated and reflected a valid JPEG image. A quick check confirmed that the file was indeed a JPEG, not a PNG.

To view the image, I used:
```
xdg-open thm.jpg
```

The image successfully opened and revealed a potential hidden directory within the target server.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%206.png)

Navigating to the newly uncovered directory led to a webpage containing a mysterious “secret” that needed to be solved. Inspecting the page source code revealed a hint on how to crack the secret.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%207.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%208.png)

Using the hint, I formulated a python script that automated the process of guessing the secret using the URL parameter `?secret=` and successfully uncovered the secret password!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%209.png)

The newly obtained password was then used with the Steghide tool to extract hidden data from the `thm.jpg` file. This process successfully uncovered a `.txt` file that contained a username!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2010.png)

This part of the lab became even trickier. Although I had uncovered both a username and a password, the initial attempt to connect via SSH was unsuccessful. After some digging around, it became clear there was another image possibly holding extra data, the very first one on the THM Madness room landing page, which many, like myself, likely overlooked.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2011.png)

After downloading the Madness image file, I used Steghide to extract any hidden contents without requiring a password. This revealed the user’s actual password.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2012.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2013.png)

Additionally, I discovered that the initial username was encoded with ROT13. Using CyberChef to decode it revealed a usable username, which allowed me to successfully attempt the SSH connection.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2014.png)

The obtained username and password were used to successfully connect to the user’s account via SSH, granting limited privileges on the server.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2015.png)

From there, the `user.txt` file was sitting within the user’s home directory for grabs.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2016.png)

Unfortunately, the user did not have root access to view the `/root` directory so our privileges needed to be escalated. The command `find / -perm -4000 -type f 2>/dev/null` was used to search the entire system for SUID files to exploit. 

Among the results, the binary `/bin/screen-4.5.0` stood out due to the existence of a known local privilege escalation exploit.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2017.png)

To execute the exploit, I saved it as a Python file and granted it execute permissions using the `chmod +x` command. Once run, the exploit granted root access, revealing the final flag. With that, the box was officially rooted and my sanity, for the most part, remained intact despite the madness.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2018.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Madness/Images/Screenshot%2019.png)

**Lessons Learned**

This room sharpened my attention to detail, teaching me to examine even seemingly irrelevant elements, like images found outside the target machine. I learned how to analyze an image’s metadata, specifically its head and tail, and how to modify an image’s header. I also became familiar with the appearance and function of ROT13 ciphers, and improved my ability to use Python to automate repetitive tasks.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Madness Room](https://tryhackme.com/room/madness)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [Exploit Database](https://www.exploit-db.com/exploits/41154)
