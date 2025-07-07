**Include (CTF) — TryHackMe Writeup**

ABOUT ME

I'm Sarath Dev. R, a cybersecurity enthusiast focused on web application security, penetration testing, and real-world offensive techniques. I'm currently building hands-on skills through platforms like TryHackMe and developing custom tools for ethical hacking and red teaming.

Connect with me on LinkedIn (www.linkedin.com/in/sarath-dev-r-b7a63331a) to follow my journey or collaborate on security-focused projects.

In this writeup, I document my process and methodology for the TryHackMe Include room. This exercise shows how I used different injection techniques to take control of a web application.

As you read through this account, think about how you can protect your own web applications from these types of attacks. The goal is to understand how these attacks work and learn how to prevent them.

In this Capture The Flag (CTF) walkthrough, we will dive into an interesting challenge on TryHackMe. The challenge revolves around exploiting a series of vulnerabilities in a web application. By applying techniques such as Server-Side Request Forgery (SSRF), Local File Inclusion (LFI), and Log Poisoning, we gain privileged access to the system. The goal is to leverage these vulnerabilities to capture the flag and understand how these attacks can be applied in a real-world context.

◈ RECONNAISSANCE

Start the machine and collect the “Target IP Address” to begin the scan.

![image](https://github.com/user-attachments/assets/e67ea1aa-01d4-4e5b-870d-0db8e7ec0958)

To get an overview of the open ports on the target system, perform an Nmap scan using the following command:

CMD:"sudo nmap -Pn -vv -T4 -n <ip>"

![image](https://github.com/user-attachments/assets/74d82d7d-a7aa-4265-ae60-791ca3781f30)

The scan detected eight open ports: 22, 25, 110, 143, 993, 995, 4000, and 50000. For this challenge, we will exclude ports 22, 993, and 995 as they are not immediately relevant. Next, scan the remaining ports one by one.

PORT 25 — SMTP

CMD: "sudo nmap -A -vv -T4 -n -p25 <ip>"

![image](https://github.com/user-attachments/assets/e10873c5-63dc-4d1f-9838-d19fa39b147a)

Port 25/tcp is open, running Postfix SMTP. The server supports commands like PIPELINING, STARTTLS, and SMTPUTF8, and can handle messages up to 10 MB. Notably, it also responds to the mail.filepath.lab command, which could indicate a custom or internal domain/mail configuration. This will be our primary focus in this room.

PORT 110— POP3

CMD: "sudo nmap -A -vv -T4 -n -p110 <ip>"

![image](https://github.com/user-attachments/assets/52272f3e-f797-4dbc-b46f-c4e4bbed79eb)

Port 110/tcp is open, running Dovecot POP3. The SSL certificate is self-signed with the common name ip-10–10–31–82.eu-west-1.compute.internal, indicating it’s part of an internal network.

PORT 143— IMAP

CMD: "sudo nmap -A -vv -T4 -n -p143 <ip>"

![image](https://github.com/user-attachments/assets/eeeba4c1-ea93-4346-a86a-858907f4175d)

Port 143/tcp is open, running Dovecot IMAP on Ubuntu.

PORT 4000— remoteanything

CMD: "sudo nmap -A -vv -T4 -n -p4000 <ip>"

![image](https://github.com/user-attachments/assets/1056fab2-637c-4e39-b530-1a376ae4c33c)

Port 4000/tcp is open, running Node.js with Express middleware. The server supports GET, HEAD, POST, and OPTIONS HTTP methods. The HTTP title is “Sign In,” indicating a login page or authentication interface. This will be our primary focus in this room.

PORT 50000— ibm-db2

CMD: "sudo nmap -A -vv -T4 -n -p50000 <ip>"

![image](https://github.com/user-attachments/assets/97c12e70-0734-4fc4-bfec-f4b06d2b8dc3)

Port 50000/tcp is open, running Apache HTTPD 2.4.41 on Ubuntu. The server supports GET, HEAD, POST, and OPTIONS HTTP methods, and the HTTP title is “System Monitoring Portal,” indicating it might be an admin interface for system monitoring. This will be our primary focus in this room.

◈ EXPLORING THE REVIEW APP

Visiting port 4000, I discovered a login page with the default credentials:

![image](https://github.com/user-attachments/assets/9beef0c5-f69d-4425-ab91-329ed360ca96)

After logging in, I saw a “Review App” interface, displaying my profile and a list of friends I could add.

![image](https://github.com/user-attachments/assets/5772416b-4bb1-4d57-b86d-cbf769c19c1d)

◈ MANIPULATING USER PREVILEGES

Clicking “View Profile”, I noticed the profile details contained an “isAdmin” field set to false.

![image](https://github.com/user-attachments/assets/e1d9a35e-dc36-4d88-a2f8-b249deef528f)

I attempted to change my age value to 20, and it worked.

![image](https://github.com/user-attachments/assets/594a7eab-94c9-4e87-b0e5-ceafef7a175c)

![image](https://github.com/user-attachments/assets/952a4ddf-48af-48e4-a3af-e7a068221dc7)

Next, I modified “isAdmin” to “true”.

![image](https://github.com/user-attachments/assets/f5702fd6-1093-4540-9225-8cb9f82df436)

Success! The admin panel was unlocked, revealing new navigation options: “API” and “Settings”.

![image](https://github.com/user-attachments/assets/f37012ca-ca9c-4f08-a58c-46e77237f919)

◈ EXPLOIT#1 SSRF

While exploring the Settings menu, I found a feature that accepts an image URL, downloads the image, and sets it as the banner. This seemed like a perfect vector for a Server-Side Request Forgery (SSRF) attack since this interacts with an external URL.

![image](https://github.com/user-attachments/assets/5ae170af-f17c-4e88-b085-95ae93d056ff)

To verify this, I set up a simple HTTP server using Python and pointed the input field to my own server’s IP address. The server made an outbound request to my server, confirming the SSRF vulnerability.

![image](https://github.com/user-attachments/assets/84b4622b-7f40-47a6-89bf-404a23dd3254)

![image](https://github.com/user-attachments/assets/f550ffb3-035b-4713-b231-04f1572a6403)

![image](https://github.com/user-attachments/assets/70575233-d32d-43cb-99a2-9db589ba8c7b)

Next, I explored the API Dashboard and found two APIs:

Internal API

Get Admins API

![image](https://github.com/user-attachments/assets/8d75f3c8-96f8-4e4b-87f4-99c9a5a389c2)

![image](https://github.com/user-attachments/assets/37cf0677-1475-46b2-8710-0deacb51ba1d)

I used SSRF via the “Admin Settings” input field to send internal requests.

![image](https://github.com/user-attachments/assets/a96585a6-bbcd-40e4-945f-c75a0b1d183e)

I got a base64 encoded string, and i decoded it using https://dencode.com/

![image](https://github.com/user-attachments/assets/bea0eb71-90a8-420b-bc64-56cc0d7c7c9f)

It reveals something that I already knew.

![image](https://github.com/user-attachments/assets/801b9083-4722-42fe-9a7f-57d02244fa60)

Then, I tried to hit the Get All Admins API.

![image](https://github.com/user-attachments/assets/0faa9071-bb77-4381-be33-367f5ef64478)

I got a base64 encoded string, and i decoded it using https://dencode.com/

![image](https://github.com/user-attachments/assets/f9e48949-7a4b-4a97-a120-f14b582ab20d)

It reveals the Review App username and password, as well as the SysMon (System Monitoring) app username and password.

![image](https://github.com/user-attachments/assets/6155068b-4aee-46c0-a59d-0b6b146f231f)

Using the leaked credentials, I attempted to log in to the Review App, but it failed.

![image](https://github.com/user-attachments/assets/49d818b4-bc9d-4abe-a634-1ec601bbfba8)

https://miro.medium.com/v2/resize:fit:720/format:webp/1*XYZKkRJ8Y9Ck9iF8irvpeQ.png

Next, I tried the System Monitoring Portal (Port 50000) and logged in using the credentials that i got before from hitting the internal API.

![image](https://github.com/user-attachments/assets/bf3b5b97-e9a0-41d9-b389-c94de45e8715)

![image](https://github.com/user-attachments/assets/5d77b271-1bbc-40f9-894a-1645fc60b302)

I was able to log in and gain full access to the system.

Don’t forget to capture the flag!

![image](https://github.com/user-attachments/assets/86c4e609-354a-41f8-85a0-66013d77b8b2)

◈ EXPLOIT#2 LFI+LOG POISONING 

While exploring the web application, I opened the source code of the page and noticed a reference to a profile image. This image was sourced from the following URL pattern:

profile.php?img=profile.png

This URL immediately raised suspicion because it appeared vulnerable to a Local File Inclusion (LFI) attack. LFI vulnerabilities occur when user input is used to include files without proper validation, potentially allowing an attacker to read sensitive files from the server.

![image](https://github.com/user-attachments/assets/75d5e571-8a86-46d9-a3cd-997f842b3d45)

The URL structure exposed a ?img= parameter, which is a typical target for LFI attacks. If the server does not adequately sanitize input, an attacker can manipulate this parameter to include files from the server's filesystem, which could lead to the exposure of sensitive files like /etc/passwd or other critical configuration files.

![image](https://github.com/user-attachments/assets/63c3e4bf-d9fc-4183-a85b-59565b4a6dd2)

To exploit the potential LFI vulnerability, I used Burp Suite Intruder to fuzz the img parameter. Fuzzing is a method of automated testing where various inputs are submitted to a web application to identify unexpected behavior or security vulnerabilities.


![image](https://github.com/user-attachments/assets/e92e53cb-1f9d-456f-8f01-c8c2a3d87d85)

I referred to the LFI payloads list from SecLists and started testing common payloads that could trigger an LFI vulnerability. I tested paths like:

....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd

This payload attempts to traverse directories using ../ (dot-dot-slash) multiple times, eventually reaching the /etc/passwd file, which contains user account details on Unix-based systems.

![image](https://github.com/user-attachments/assets/3b986fe6-85b7-4830-8269-32fecbe44739)

To my satisfaction, the payload was successful, and it revealed the /etc/passwd file. This confirmed that the application is indeed vulnerable to LFI, as it was able to read system files without proper input sanitization.

![image](https://github.com/user-attachments/assets/0d9a007c-efc9-4301-8510-98dfc5aa53f8)

The output of the /etc/passwd file exposed the following users:

joshua

charles

This discovery provided useful information, as these could potentially be valid user accounts for further exploitation.

![image](https://github.com/user-attachments/assets/fcd8e6e6-01d4-455b-b4b0-0e88fab351a1)

Next, I attempted to access other files on the system by modifying the payload to include different paths. For instance, I tried navigating to:

....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//var/www/html

However, this attempt failed. It seems the file path was either restricted or did not exist, which meant I was unable to access the desired directory.

![image](https://github.com/user-attachments/assets/706629aa-aa63-464b-9a1b-918ffecc2887)

Not giving up, I shifted my focus to the log files. Log poisoning involves manipulating logs to execute arbitrary commands or expose sensitive information. I tried the following path to access the mail logs:

....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//var/log/mail.log

When I accessed this log file, I discovered that it contained valuable information, such as:

Logs of incoming and outgoing emails

Server-side events that could give insight into how the application functions

![image](https://github.com/user-attachments/assets/a6e2948f-f263-4a84-876e-915a7c5b2262)

As part of the next phase, I decided to try sending an email to joshua using the Telnet protocol. Telnet is a simple, text-based protocol often used for sending data over a network, and in this case, I used it to simulate the process of sending an email.

I connected to the SMTP server (on port 25) and used the Telnet interface to send an email to the joshua account. This action is intended to trigger a log entry in the mail.log file.

![image](https://github.com/user-attachments/assets/703533eb-747c-40f3-a2a0-234ddcf85098)

After sending the email using Telnet, I sent another request from Burp Suite to confirm that the email was correctly logged. Upon inspecting the mail.log file again, I found that the log entry indeed showed the email being sent to joshua.

![image](https://github.com/user-attachments/assets/db876b40-0025-42d1-9fb4-0054408cdf80)

Now that I knew the mail.log file was being populated with logs from the email interactions, I turned my attention to log poisoning. According to an article from HackingArticles, when you have access to a log file like mail.log due to an LFI vulnerability, the log file typically has both read and write permissions. This means we can inject malicious code into the log file, which will be executed later when the log file is accessed by the server.

Here’s how the technique works: when the server processes the mail logs, it generates entries for each mail sent. By leveraging this feature, we can inject malicious PHP code into the log file by sending a crafted email through Telnet, specifically using the RCPT TO field. The payload I injected was:

<?php system($_GET['cmd']); ?>

This PHP code is designed to execute commands passed through the cmd query parameter in the URL, making it a simple but effective way to gain command execution on the server.

The RCPT TO field in the email request was used to send this PHP code, and while the server responded with a 501 5.1.3 Bad recipient address syntax error (indicating an incorrect email format), the actual goal was to inject this code into the mail.log file, not to send a legitimate email.

The key part to note is that while the server was expecting an email address, I injected OS commands disguised as email recipient addresses. This trick allowed the PHP code to be written to the mail.log file as part of the logging process. The code was now stored in the mail.log, and it could be executed when accessed later.

![image](https://github.com/user-attachments/assets/291d0f8b-0252-4016-b8d8-d13ae25bb15e)

After successfully injecting the PHP code into the mail.log file, I decided to test the attack again. I returned to Burp Suite and hit the target again. This time, I carefully monitored the mail.log entries and found that the RCPT command — used to send the malicious payload — was logged at the end of the request. This confirmed that the RCPT TO field was being processed as part of the email transaction, and the PHP payload had been successfully logged.

![image](https://github.com/user-attachments/assets/790fc5e2-0162-48fb-b419-981787ba2e00)

Now that the PHP payload was injected into the mail.log file, I proceeded to execute a command by adding a cmd parameter to the URL.

I crafted the following request:

profile.php?img=....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//var/log/mail.log&cmd=whoami

When the request was sent, the whoami command was passed as a GET parameter and executed through the injected PHP code in the mail.log file. The response confirmed that the command had executed successfully, and it returned the result of “www-data”.

![image](https://github.com/user-attachments/assets/2d45fec2-0a6d-47ee-8cdc-4291c4a766e7)

I decided to probe further by listing the contents of the /var/www/html directory to find the flag. Heck yeah I got it.

![image](https://github.com/user-attachments/assets/445bacde-bce0-4ca3-a84f-e1e1c527b8be)

I used the cat command to open it. Voila!

![image](https://github.com/user-attachments/assets/a92a052a-15f9-4534-92ee-04cd97dff4d4)

◈ CONCLUSION

In this write-up, we’ve walked through several vulnerability exploitation techniques, from basic reconnaissance with Nmap to more advanced attacks like SSRF and LFI.

















































