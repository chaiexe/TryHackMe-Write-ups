**<p align="center">Hacking Challenge Day 8 - Topic: Software and Data Integrity Failures</p>**
---
[![Day 8 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%208%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Completed: 6/27/2025**

🔒 Topic: Software and Data Integrity Failures (OWASP A08 - 2021)

<p align="center">+++++++++</p>

A **Software and Data Integrity Failure** is a web application vulnerability that occurs when an application relies on untrusted sources for updates, plugins, dependencies, or critical data without verifying their integrity. In other words, the web application assumes outside sources are safe, even though attackers could have secretly changed them.

**Why it matters:** Attackers can exploit these weaknesses to spread malicious code through the very tools developers rely on. This often leads to large-scale compromises, where many systems are affected at once.

**Analogy:** Think of it like downloading an update for your phone, but the update isn’t checked to make sure it’s really from the official source, if it’s fake, your whole device could be at risk.

<p align="center">+</p>

**Real-World Example**

Imagine a web application that loads a JavaScript file from a third-party Content Delivery Network (CDN). If the app doesn’t verify the file’s integrity with a method like Subresource Integrity (SRI), attackers can tamper with it. If the CDN is compromised, every user visiting the site unknowingly runs the malicious script.

This is how incidents like SolarWinds happened. A trusted software update was secretly modified and used to infiltrate government and private networks worldwide.

<p align="center">+</p>

**Remediations:**

- Use digital signatures to verify software updates and packages.

- Implement cryptographic hash validation for critical data and dependencies.

- Enable Subresource Integrity (SRI) for third-party scripts and assets.

- Secure CI/CD pipelines by validating inputs and restricting unauthorized changes.

- Avoid untrusted or outdated components in your application stack.

- Enforce strong access controls on deployment and build systems.

<p align="center">+++++++++</p>

**<p align="center">✨Research Deep-Dive✨</p>**

What is **Subresource Integrity (SRI)**?

Modern browsers support a security mechanism called **Subresource Integrity (SRI)**, which allows developers to specify a cryptographic hash alongside the URL of an external resource (like a JavaScript library). This ensures that the resource is only executed if the file's hash matches the expected value, protecting against tampering or CDN compromise.

You write an HTML `<script>` tag in your website’s code that references the external file by its URL, and you add the hash as an attribute to that tag. It looks something like this:

```
<script 
  src="https://cdn.example.com/library.min.js" 
  integrity="sha384-abc123..." 
  crossorigin="anonymous">
</script>

```
- `src:` Where the browser gets the file from (the CDN link).

- `integrity`: This is where you put the hash, it tells the browser what the file is supposed to look like.

- `crossorigin="anonymous"`: This is often used with SRI to avoid issues with cross-origin requests.

If the file is altered (even a single character), the hash won't match, and the browser will block it from loading.

**Note:** When viewing a webpage’s source code, look for `<script>` or `<link>` tags that load files from external sources (like CDNs). Ex:

```
<script src="https://cdn.example.com/library.min.js"></script>
```
Check if the `integrity` attribute is missing.

To securely include an external library in your HTML, always use Subresource Integrity (SRI) by adding an `integrity` hash. This ensures that if an attacker modifies the library, any visitor to your website will not run the tampered version, protecting your users from malicious code.

Without integrity checks, the CDN could be compromised and malicious code would run without being detected.

<p align="center">+++++++++</p>

**Notes:**

When a user logs into an application, they will be assigned some sort of session token that will need to be saved on the browser for as long as the session lasts. This token will be repeated on each following request so that the web application knows who we are. These session tokens can come in many forms but are usually assigned via cookies. **Cookies** are key-value pairs that a web application will store on the user's browser and that will be automatically repeated on each request to the website that issued them.

To help prevent cookie tampering, **JSON Web Tokens (JWTs)** were introduced. JWTs are simple tokens that store key-value pairs and include a built-in integrity check. This means the server can trust that the data inside the token hasn’t been altered by the user. The token is signed, so if someone tries to change its contents, the signature won’t match and the token will be rejected.

A **JWT (JSON Web Token)** has three parts:

1. Header – indicates this is a JWT and specifies the signing algorithm used (like HS256).

2. Payload – contains the actual data, such as user information, in key-value pairs.

3. Signature – acts like a security seal, proving the data hasn’t been changed.

The signature is made using a secret key that only the server knows. If someone tries to change the data in the payload, the signature won’t match anymore, and the server will know the token was tampered with. This keeps the token’s data safe and trustworthy.

**Note:** “The secret key is held by the server only, which means that if you change the payload, you won't be able to generate the matching signature unless you know the secret key.” - THM

<p align="center">+</p>

Using the AppDevTools Online Base64 Encoder/Decoder website to decode the following JWT header and payload:
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0IiwiZXhwIjoxNjY1MDc2ODM2fQ.C8Z3gJ7wPgVLvEUonaieJWBJBYt5xOph2CpIhlxqdUw
```
The decoded results are:
```
{"typ":"JWT","alg":"HS256"} | {"username":"guest","exp":1665076836} | 
Æwð>K¼E(¨%`IyÄêaØ*H\juL
```
<p align="center">+++++++++</p>

**<p align="center">💾Data Integrity Failures (Challenge)💾</p>**

Navigating to the URL `hxxp[:]//10.10.175.35:8089/ ` opens to the **Cookies4all** web application. Attempting to log in with the username and password `admin` results in the message: “Invalid Credentials. You can also login as "guest" with password "guest".

The message confirms the valid credentials for the guest account.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/OWASP-Top-10-2021/08-Software-and-Data-Integrity-Failures/Images/Screenshot%201.png)

Successfully logging in with the guest credentials displays the message:

“Hello guest. Only the admin user is allowed to get the flag!”

Using the browser’s **Developer Tools**, the **Storage** tab confirms that a JWT is stored as a session cookie under the name `jwt-session`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/OWASP-Top-10-2021/08-Software-and-Data-Integrity-Failures/Images/Screenshot%202.png)

Using the AppDevTools Online Base64 Encoder/Decoder website to reconstruct the Header and payload portion of the JWT session cookie resulted in the following:

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/OWASP-Top-10-2021/08-Software-and-Data-Integrity-Failures/Images/Screenshot%203.png)

Reloading the `hxxp[:]//10.10.175.35:8089/flag` page with the modified JWT session cookie successfully revealed the admin flag.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Red-Team/OWASP-Top-10-2021/08-Software-and-Data-Integrity-Failures/Images/Screenshot%204.png)

**Lessons Learned:** 

This challenge demonstrated how weak or missing signature validation in JWTs can easily lead to authentication bypass and privilege escalation. By modifying the JWT header, payload, and removing the signature, I was able to impersonate an admin user and access sensitive content. It highlights the importance of verifying JWT signatures on the server side to ensure token integrity and protect against tampering.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in
understanding the vulnerability.

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
- [AppDevTools](https://appdevtools.com/base64-encoder-decoder)
