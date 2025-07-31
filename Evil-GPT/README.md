**<p align="center">TryHackMe: Evil-GPT</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Evil-GPT/Images/Room%20Icon.png" alt="image alt" width="180" />
</p>

[![Day 23 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2023%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** LLMs (Large Language Models) hacking

**Completed:** 7/31/2025

“Cipher’s gone rogue—it’s using some twisted AI tool to hack into everything, issuing commands on its own like it’s got a mind of its own. I swear, every second we wait, it’s getting smarter, spreading chaos like a virus. We’ve got to shut it down now, or we’re all screwed.”

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

In penetration testing (pentesting), LLMs (Large Language Models) are AI models trained on massive datasets of text and code, capable of understanding and generating human-like text. In the context of cybersecurity, LLMs are increasingly being explored for their potential to automate or augment various pentesting tasks. They can be used to analyze code for vulnerabilities, generate attack payloads, craft phishing emails, and even design entire penetration testing strategies

<p align="center">+++++++++</p>

Starting of with an Nmap scan, I discovered one open port:

- 22/tcp SSH OpenSSH 8.9p1 Ubuntu 3ubuntu0.11 (Ubuntu Linux; protocol 2.0)

Since common web ports like 80 or 443 were not open, there's no need to attempt accessing the IP via a web browser.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Evil-GPT/Images/Screenshot%201.png)

Following the room’s instructions to connect to the target machine using the `nc 10.201.113.240 1337` command connects to what seems to be an AI command executor.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Evil-GPT/Images/Screenshot%202.png)

Write-up pending ~
