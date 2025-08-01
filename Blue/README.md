**<p align="center">TryHackMe: Blue</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Blue%20Icon.gif" alt="image alt" width="100" />
</p>

[![Day 22 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2022%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed:** 7/30/2025

**Topic:** Windows Priv-Esc

“Deploy & hack into a Windows machine, leveraging common misconfigurations issues.”

📢 Hacking into Windows via EternalBlue, Blue badge included after completing this room‼️

<p align="center">+++++++++</p>

Starting the room off with an `Nmap -sV -Pn` scan uncovered three open ports with a port number under 1000:

- 135/tcp   open  msrpc        Microsoft Windows RPC
- 139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
- 445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)

Since common web ports like 80 or 443 were not open, there's no need to attempt accessing the IP via a web browser. The presence of port 445 (SMB) immediately stands out as a potential attack vector, especially given the room's name (Blue), hinting at the EternalBlue (MS17-010) vulnerability.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%201.png)

Using the search function in Metasploit, I looked for modules related to the EternalBlue (MS17-010) vulnerability. The module `exploit/windows/smb/ms17_010_eternalblue` appeared to be the most accurate and appropriate choice for targeting this SMB flaw. 

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

**The SMB Flaw (MS17-010 - EternalBlue)**

The flaw lies in the Server Message Block (SMBv1) protocol. A Windows service used for file sharing, printer access, and network communication over ports 445 and 139.

In unpatched versions of Windows, the SMBv1 implementation improperly handles specially crafted packets, which allows an attacker to remotely execute code without authentication, allowing full control over the system. The vulnerability is identified as MS17-010, and the exploit leveraging it is known as **EternalBlue**.

So in essence, the flaw is a critical memory corruption vulnerability in SMBv1.

<p align="center">+++++++++</p>

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%202.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%203.png)

Using the show options command, I reviewed the required parameters for the exploit. A few values needed to be adjusted to match the target machine’s IP address, my local IP (LHOST), and the selected payload.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%204.png)

Although a default payload was already configured for the module, I explicitly set the payload again to follow the TryHackMe instructions precisely. Once all configurations were in place, I executed the exploit, successfully gaining a basic Windows shell on the target machine.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%205.png)

After gaining the initial Windows shell, I backgrounded the session using the `CTRL+Z` command which allowed me to search for the Metasploit module:
```
post/multi/manage/shell_to_meterpreter
```

This post-exploitation module is used to upgrade a basic shell into a more useful Meterpreter session allowing better control, features, and stability.

Before executing the module, I adjusted the options:

- `LHOST`: Set to my local machine's IP

- `SESSION`: Set to the ID of the original shell session (in this case, session 1)

Running the module successfully created a second session, this time with a fully functional Meterpreter shell.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%206.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%207.png)

I used the `sessions` command to list all active sessions, then connected to the newly spawned Meterpreter shell by running `sessions -i 2`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%208.png)

Using the `ps` command listed all the currently running processes to find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM. 

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%209.png)

Process ID 1304 stood out as it was a command-line instance running under the NT AUTHORITY\SYSTEM account, indicating a potential privilege escalation path.

Using the `migrate` command in Meterpreter, I successfully migrated the session into this process. The `migrate` command allows the current session to move into a different process on the target system, in this case, one with SYSTEM-level privileges

With SYSTEM privileges secured, I proceeded to dump the stored password hashes using the `hashdump` command, revealing the non-default user on the machine `Jon`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%2010.png)

CrackStation was used to successfully uncover the plaintext password from the hash dump.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%2011.png)

To begin the flag collection, I navigated to the `C:\` directory, where the `dir` command revealed `flag1.txt`. I used `cat` to display its contents.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%2012.png)

Since I’m more familiar with typical flag locations on Linux machines, I took a moment to research where flags are commonly stored on Windows boxes. Some of the commonly used directories include:

- C:\Users\ [username]\Desktop

- C:\Windows\System32\config

- C:\ProgramData

In this case, the second flag was located in the `C:\Windows\System32\config` directory

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%2013.png)

Concluding the search, the last flag was located in the user's `Documents` folder at `C:\Users\Jon\Documents`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue/Images/Screenshot%2014.png)

<p align="center">+</p>

**Lessons Learned**

This Windows-based box provided a refreshing challenge and expanded my comfort zone beyond Linux environments. I gained practical experience exploiting the EternalBlue (MS17-010) vulnerability, understanding how to migrate sessions within Meterpreter for privilege escalation, and navigating the Windows file system to uncover hidden flags. I also learned how to extract and crack password hashes using hashdump and CrackStation, which reinforced the importance of post-exploitation enumeration. Overall, this lab deepened my understanding of how Windows handles processes, permissions, and privilege levels, and gave me more confidence in approaching real-world Windows privilege escalation scenarios.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Blue Room](https://tryhackme.com/room/blue)
- [Crackstation](https://crackstation.net/)
