**TryHackMe: Injectics Walkthrough**

ABOUT ME

I'm Sarath Dev. R, a cybersecurity enthusiast focused on web application security, penetration testing, and real-world offensive techniques. I'm currently building hands-on skills through platforms like TryHackMe and developing custom tools for ethical hacking and red teaming.

Connect with me on LinkedIn (www.linkedin.com/in/sarath-dev-r-b7a63331a) to follow my journey or collaborate on security-focused projects.

In this writeup, I document my process and methodology for the TryHackMe Injectics room. This exercise shows how I used different injection techniques to take control of a web application.

As you read through this account, think about how you can protect your own web applications from these types of attacks. The goal is to understand how these attacks work and learn how to prevent them.

◈ ENUMURATION

I started with scanning open ports of the target.

![image](https://github.com/user-attachments/assets/78938a59-4e10-4d35-907a-e492fb18c2c4)

22, and 80 are open.

Visited port 80:

![image](https://github.com/user-attachments/assets/44fdbaee-c734-4ac9-a9c8-fc2bbd226960)

The webpage welcomed me.

![image](https://github.com/user-attachments/assets/4f00d1bd-2c12-4323-ac60-57225482dc6a)

The website had a login page and It was also possible to login as an admin.

As usual, fuzzing directories may be a good idea. In this walkthrough, I decided to do it via dirsearch, because it uses some additional extensions on default.

![image](https://github.com/user-attachments/assets/e7d534e6-4ec0-433d-84a6-4d8f0e84fc8a)

![image](https://github.com/user-attachments/assets/6ba1b4ec-fe84-4327-aade-1042de6161d7)

Pay attention to the composer.json, phpmyadmin.

![image](https://github.com/user-attachments/assets/3cb558d9-bafd-479e-afe7-a82969ae34fa)

![image](https://github.com/user-attachments/assets/1816fe7c-544a-4103-aac9-009304756096)

Twig is a template engine for PHP programming language.

The website also uses MySQL database.

To bypass the login page, I have to find a valid credential, so I decided to start viewing the page source.

![image](https://github.com/user-attachments/assets/41729dba-ca13-4f43-93dd-b64658eccd8e)

The page source contains log file, which is a hint for me.

![image](https://github.com/user-attachments/assets/7c137a67-48cd-402a-bb80-1b0722738ad7)

Oops, credentials are disclosed in the mail, but mail also tells that If anything happens to the ‘users’ database, the following accounts would be enabled with the default passwords. So, I had to find another way to get in.

◈ AUTHENTICATION BYPASS VIA SQLi

First, I tried basic SQLi payloads for bypassing the login page, but they didn’t work for me.

![image](https://github.com/user-attachments/assets/68d189b9-7823-480f-b5e7-6bc68f895ecb)

In this case, fuzzing input might be a good way.

So, I found a list of payloads on the GitHub and it was very useful.

Time to find a useful payload to bypass the authentication mechanism.

After capturing the login page request, I sended it to the Intruder, then selected username input and added the list I found before.

![image](https://github.com/user-attachments/assets/9e19cf20-cdb4-41b8-acfd-23b64ad10319)

Then, started the fuzzing.

![image](https://github.com/user-attachments/assets/23091077-3e3e-4191-a054-43236d3d4420)

After some time, I got the successful payload.

For finding an alternative way, I also tried it for password input, but didn’t find any successful payload.

◈ LOGGING INTO ADMIN PANEL

After visiting the dashboard.php, it was clear that I should try SQLi on the update table feature.

![image](https://github.com/user-attachments/assets/63fcfd19-9f56-4226-b7a4-7252ad125134)

I added a simple SQLi payload:

1’ SELECT 1;

![image](https://github.com/user-attachments/assets/9f047d53-6e41-466d-9669-af7b21f82f73)

![image](https://github.com/user-attachments/assets/45b2b4a9-3215-4d68-8f40-16e70d336f40)

It returned an error because of validation, so, I changed the payload:

1; SELECT 1;

![image](https://github.com/user-attachments/assets/4f7c2229-313b-4d49-b52d-ee3202604e52)

![image](https://github.com/user-attachments/assets/15dd249b-d00e-4acb-8de8-3d7a847001d3)

After updating successfully, the website redirected me to the dashboard.

It was interesting, but I couldn’t enter to the system as an admin by performing this.

Then I remembered the mail.log I found in source code. Time to delete the ‘users’ table. By doing this, I could enter the admin panel with default creds.

![image](https://github.com/user-attachments/assets/13591b72-d20d-4f28-b1dc-d82d537843b7)

![image](https://github.com/user-attachments/assets/6210c846-e189-48fb-8b4d-345bfdf0f436)

After some time, I tried to enter the admin page.

![image](https://github.com/user-attachments/assets/03207f51-7a19-4ab7-aa8a-5c4e7af385cc)

![image](https://github.com/user-attachments/assets/1e4c6d8a-6b0d-4624-bc1c-bd075664d624)

And I also got the first question’s flag.

AND I'M SORRY FIND THE ANSWER BY YOURSELF...

◈ LAST FLAG

When fuzzing for the hidden directories, I found a one named flags. Maybe getting the last flag was easy than I thought.

![image](https://github.com/user-attachments/assets/0114b02f-ddae-46ba-9ea3-2d1a4b8f155a)

Unfortunately, it wasn’t.

Okay, the dashboard.php has a “Profile” directory and it allowed me to update user’s informations.

![image](https://github.com/user-attachments/assets/6f03841f-4fa3-44be-8a73-f1ed855e6cb4)

![image](https://github.com/user-attachments/assets/107544af-75f9-402e-8392-d4b43901df78)

Here, I tried a payload to identify a vulnerability called Server-Side Template Injection. Why?

Because I discovered composer.json at the beginning and I already knew that the website was using template engine.

The payload I regularly use to find SSTI — "${{<%[%’”}}%\."

![image](https://github.com/user-attachments/assets/a4225c5f-cf14-4ba2-9132-269ffebc7658)

![image](https://github.com/user-attachments/assets/24143336-b750-48c0-90b6-b44bc62a4a9a)

I knew the website was using Twig, so I submit the first name with the following payload — {{7*7}}.

If it returns 49, then the website has SSTI vulnerability.

![image](https://github.com/user-attachments/assets/d6e85e81-b557-42b3-9cf1-bdf44fb7e792)

![image](https://github.com/user-attachments/assets/81efd920-6e7e-41a8-9dc3-4683b27ad866)

49 means it was time to inject system commands.

![image](https://github.com/user-attachments/assets/26012180-b3df-45d4-819b-811229a67215)

![image](https://github.com/user-attachments/assets/61c7c8ac-de47-4c8e-a4d9-c2ff3a2f6f6b)

Unfortunately, it returns an error. I tried to simplify the payload.

"{{[‘id’,””]|sort(‘passthru’)}}"

![image](https://github.com/user-attachments/assets/c5e34b5b-978f-4563-bbff-82ccf83f9187)

![image](https://github.com/user-attachments/assets/71f4149d-a04f-4338-a398-49f216da7efc)

And it worked.

I changed the commands to list what was hiding inside the /flags.

![image](https://github.com/user-attachments/assets/9a135692-6dcc-441c-8174-b214f846aeeb)

![image](https://github.com/user-attachments/assets/7d751e85-5a08-4e73-a6fa-4590660d573f)

And the last flag was there.

![image](https://github.com/user-attachments/assets/b73e9e56-3af5-4d3a-ba83-023689a932c7)

![image](https://github.com/user-attachments/assets/a06895e3-358a-4c6a-b89a-1b119db927df)

![image](https://github.com/user-attachments/assets/6a42c60e-ad32-4b05-a702-de849024904b)

After submitting the flag, room was completed.

**THANK YOU ALL**



































