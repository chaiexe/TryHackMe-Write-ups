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

![Alt text](1)
