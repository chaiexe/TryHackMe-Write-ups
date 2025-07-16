**<p align="center">TryHackMe: PassCode</p>**
---

<p align="center">
  <img src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/PassCode/Images/THM%20PassCode%20Image.png" alt="image alt" width="180" />
</p>

[![Day 16 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2016%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting 

**Completed:** 7/14/2025

“We may have found a way to break into the DarkInject blockchain, exploiting a vulnerability in their system. This might be our only chance to stop them—for good.”

<p align="center">+++++++++</p>

Beginning with an Nmap scan on the target IP `10.10.224.251` revealed two open ports:

- 22 (SSH)
- 80 (HTTP)

Visiting the website opens the Blockchain Challenge homepage. According to the website, the goal is to have the `isSolved()` function return true.

Screenshot 1

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

**What is Blockchain?** - In simple terms, blockchain is a way of storing information in a digital ledger that is shared across a network of computers. Every time someone writes something new, everyone’s ledger updates so they all have the same information.

**What is a Blockchain ledger?** - This ledger is designed to be secure, transparent, and difficult to alter, making it suitable for recording transactions or other data. Think of it like a shared, digital notebook that everyone on the network can see, but no one can secretly change.

**What is Ethereum?** - Ethereum is a blockchain-based platform that lets people build and use decentralized applications (called dApps). These apps don't rely on a central server, but instead run on a distributed network of computers. 

Essentially, a giant shared decentralized computer in the cloud that anyone can use, but no one can fully control.

- No one can delete or tamper with a smart contract once deployed

- It's transparent: anyone can read and verify the code

- It removes the need for trust in third parties (you trust the code instead)

<p align="center">+++++++++</p>

Given commands to try:
```
root@attacker:~# RPC_URL=http://TARGETIP:8545
root@attacker:~# API_URL=http://TARGETIP
root@attacker:~# PRIVATE_KEY=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.private_key")
root@attacker:~# CONTRACT_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".contract_address")
root@attacker:~# PLAYER_ADDRESS=$(curl -s ${API_URL}/challenge | jq -r ".player_wallet.address")
root@attacker:~# is_solved=`cast call $CONTRACT_ADDRESS "isSolved()(bool)" --rpc-url ${RPC_URL}`
root@attacker:~# echo "Check if is solved: $is_solved"
Check if is solved: false
       
```
<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

Researched Questions: What is cast call, what does it do, and why is it needed here?

x

<p align="center">+++++++++</p>

TBD....
