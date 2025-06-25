**<p align="center">Hacking Challenge Day 4 - Topic: Insecure Design</p>**
---
![Day 4 of 30 – Hack Documentation Challenge](https://img.shields.io/badge/Day%204%20of%2030-Hack%20Documentation%20Challenge-crimson?style=for-the-badge&logo=tryhackme)

**Completed: 6/19/2025**

🔒 **Topic:** Insecure Design (OWASP A04 - 2021)

🎯 **Objective:** Continue progressing through the OWASP Top 10 - 2021 room on TryHackMe

<p align="center">+++++++++</p>

**Insecure design** happens when the foundation or structure of an application is flawed in a way that creates security risks. It’s not about coding mistakes or misconfigurations, but rather about poor planning or missing safeguards built into the system from the beginning.
It’s like building a house without locks on the doors. The locks were never broken, but no one ever thought to add them in the first place.

**Remediation:**

Because insecure design issues are introduced early in the development process, fixing them often means going back and reworking the affected part of the application entirely. This makes them harder to fix compared to typical coding bugs. The best way to prevent these types of vulnerabilities is by doing proper threat modeling during the early stages of development.

<p align="center">+++++++++</p>

**<p align="center">Insecure Design (Challenge)</p>**

**Objective:** Assess the target web application to identify an insecure design vulnerability within the password reset mechanism and gain access to Joseph’s account.

<p align="center">+</p>

Engaging with the target IP at `hxxp[:]//10.10.147.180:85` loads the TryHackMe File Server login page.

While the user’s name is known, further testing is needed to determine whether it is also used as the login username. The *Forgot my password…* feature will be explored to assess its behavior and identify any potential weaknesses in the password reset mechanism.

![Alt text](1)

Interacting with the *Forgot my password…* link redirects to `hxxp[:]//10.10.147.180:85/resetpass1.php`, where the user is prompted to enter their username before proceeding.

![Alt text](2)

The username Joseph was tested. Clicking *Continue* updated the URL to `hxxp[:]//10.10.147.180:85/resetpass2.php`. Using the dropdown menu, the security question *“What’s your favorite colour?”* was selected, and the answer *red* was submitted.

![Alt text](3)

This attempt updated the URL to `hxxp[:]//10.10.147.180:85/resetpass2.php?err=1`, prompting the message:
```
“Error: Incorrect answer! Try again.”
```

Following the failed attempt, the password reset form reverted to the default question: *“What’s your mother’s sister’s son’s nephew’s neighbour’s friend name?”*, suggesting this is the fallback state of the page regardless of the question previously selected.

![Alt text](4)

A second attempt was made to answer the default security question, *“What’s your mother’s sister’s son’s nephew’s neighbour’s friend name?”* Regardless of the input provided, the URL remained unchanged as `hxxp[:]//10.10.147.180:85/resetpass2.php?err=1`, and the same error message persisted. This suggests that either the answer was incorrect, or the application is hardcoded to return the same response for all invalid submissions.

![Alt text](5)

It was also observed that the password reset security answer field accepted any input without validation. Even submitting a single character such as a period (.) resulted in the same generic error message, *“Error: Incorrect answer! Try again.”* This indicates that the application does not enforce input format or content validation for the security question’s answer.

![Alt text](6)

Testing the URL parameters by appending `user=joseph&answer=red` did not bypass the validation. The web page returned the same error message and reverted to the default password reset page, indicating that changing the URL parameters is ineffective.

![Alt text](7)

Manually navigating to `resetpass3.php` redirected back to the initial password reset page at `hxxp[:]//10.10.147.180:85/resetpass1.php.` This behavior suggests that the server enforces step-by-step access control, preventing users from skipping directly to the final step without completing the previous stages. It's likely that the server uses a session variable or some backend check to confirm the security question was answered before allowing access to the final reset page.

While this shows that some basic step tracking exists, the overall identity check still appears weak, especially if there’s only one correct answer hard-coded into the system. That makes guessing the right security answer the main way to reach `resetpass3.php`.

Additionally, submitting a random username such as "x" returned the error message:
```
"Error: the user doesn't exist!"
```
In contrast, submitting "Joseph" didn’t trigger the same error, which confirms that the username exists on the server.

![Alt text](8)

The application relies solely on a security question to verify a user’s identity during the password reset process. If the submitted answer is correct, the system assumes the requester is Joseph without any additional verification. This highlights a serious insecure design flaw: the application validates identity using only a single, easily guessable value, without any support from email confirmation, authentication tokens, or multi-factor verification.

Referring to the THM hint, *“Is there any security question that can be easily guessed?”*, the question “What’s your favorite colour?” stood out as the easiest of the three security questions to answer. Based on that, a range of common colors was tested as possible answers.

After testing *red*, the next attempt using *green* was successful, resulting in a redirect to `hxxp[:]//10.10.147.180:85/resetpass3.php` and revealing the message:

```
"Success: The password for user joseph has been reset to 9enhrKJ1uk4MNj."

```

![Alt text](9)

The password `9enhrKJ1uk4MNj` was then used to attempt a login to Joseph’s account.

**Note:** The username field is case sensitive.

Logging in with the correct credentials resulted in a successful authentication, redirecting to `hxxp[:]//10.10.147.180:85/myfiles.php`. This page displayed a document titled note.txt, which contained the message:

```
“Remember to move private files out of the server!”
```

![Alt text](10)



![Alt text](11)

Clicking on the “Private” tab revealed a document named `flag.txt`, which contained the TryHackMe flag required to complete the task.

![Alt text](12)

![Alt text](13)

**Lessons Learned:**

This lab was quite challenging and made me pause to think deeply. It emphasized how insecure design vulnerabilities often stem from fundamental flaws in application logic rather than just coding errors. The experience reinforced the value of careful testing and thoughtful analysis when evaluating authentication mechanisms.


<p align="center">+++++++++</p>

🔒*Out of respect for the learning experience, I’ve chosen not to share the final answers directly. Instead, I’ve documented my full process to support both others and myself in understanding the vulnerability.*

**Resources**:
- [TryHackMe's OWASP Top 10 - 2021 Room](https://tryhackme.com/room/owasptop102021)
