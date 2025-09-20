**<p align="center">TryHackMe: DarkMatter</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Room%20Icon.png" alt="image alt" width="140" />
</p>

**Completed:** 9/1/2025 

**Objective:** Practice exploiting a weak RSA implementation to recover the private key and decrypt files affected by ransomware.

“The Hackfinitiy high school has been hit by DarkInjector's ransomware, and some of its critical files have been encrypted. We need you and Void to use your crypto skills to find the RSA private key and restore the files. After some research and reverse engineering, you discover they have forgotten to remove some debugging from their code. The ransomware saves this data to the tmp directory.

Can you find the RSA private key?”

<p align="center">+++++++++</p>

Beginning with an `nmap -sV` scan revealed 3 open ports:
- 22 (SSH)
- 80 (HTTP)
- 5901 (VNC)

Due to the challenge being about a ransomware attack, there was no need to navigate to the webpage on port 80, as there wasn't anything there anyway.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%201.png)

On the lab workstation appeared a timed ransomware note window left by the attackers named DarkMatter Gang which included that they wanted money in BTC along with a threat of permanent data loss.
In the window is an input field for the decryption key, and since we are the specialists, the goal is to work out a way to get access to the decryption key to free the workstation.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%202.png)

Opening up the terminal, I confirmed what directory I was currently in, navigated to the `/tmp` folder, and listed its contents.

Interesting found files:
- `encrypted_aes_key.bin`: Likely contains the AES symmetric key, but is encrypted with the RSA public key. Once I recover the RSA private key, I should be able to decrypt this.

- `public_key.txt`: RSA public key used to encrypt the AES key.


**Facts:** after gaining the private RSA key I should be able to decrypt the `.bin` file to obtain the AES key, which would allow me to decrypt the ransomware-encrypted files.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%203.png)

Using the encrypted data from the `public_key.txt` file: 

```
cat public_key.txt 
n=340282366920938460843936948965011886881
e=65537
```

I used https://www.dcode.fr/rsa-cipher to decipher the rsa public key which resulted in:

```
e    65537
n    340282366920938460843936948965011886881
d    196442361873243903843228745541797845217
p    18446744073709551533
q    18446744073709551557
φ    340282366920938460807043460817592783792
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%204.png)

Since `d` (the private exponent) is `196442361873243903843228745541797845217`, that’s our ticket in.

Now that we have the private key value, I used it to successfully unlock the system.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%205.png)

Inserting the decryption key unlocked two school documents.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%206.png)

The document `student_grades.docx` included the needed THM flag to complete the room!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/DarkMatter/Images/Screenshot%207.png)

**Lessons Learned**

This lab demonstrated how ransomware can compromise a system and highlighted the importance of understanding encryption mechanisms. By analyzing leftover debugging data and recovering the RSA private key from the `/tmp` folder, I learned practical steps to obtain the decryption key and restore files, emphasizing the value of careful forensic analysis and secure key management in preventing such attacks.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's DarkMatter Room](https://tryhackme.com/room/hfb1darkmatter)
- [Cipher Decoder](https://www.dcode.fr/rsa-cipher)
