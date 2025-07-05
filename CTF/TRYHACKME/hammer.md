**HAMMER CTF TRYHACKME**

ABOUT ME

I'm Sarath Dev. R, a cybersecurity enthusiast focused on web application security, penetration testing, and real-world offensive techniques. I'm currently building hands-on skills through platforms like TryHackMe and developing custom tools for ethical hacking and red teaming.

Connect with me on LinkedIn (www.linkedin.com/in/sarath-dev-r-b7a63331a) to follow my journey or collaborate on security-focused projects.

◈ ROOM OBJECTIVE

The “Hammer” room on TryHackMe is a Capture The Flag (CTF) challenge that typically involves bypassing authentication and achieving Remote Code Execution (RCE) to obtain two objective as flags.

◈ Nmap Scan

Let’s start with scanning all ports

◦ CMD : " nmap -a -sV -T4 10.10.26.56 "

![image](https://github.com/user-attachments/assets/54dc5051-23a9-4799-a387-9ae6c6cd5bed)

We find two ports in Open State : 22 and 1337

Let’s check what we find on port 1337

![image](https://github.com/user-attachments/assets/13e22ad6-ddc6-4fc7-9b84-703ed5791a3f)

There is a login page @ port 1337 and we don’t really know the username and password , using default credentials like — admin:admin , admin:password , etc — were of no use and returned as login failure

![image](https://github.com/user-attachments/assets/1c6a63ef-5245-4201-bb92-52245bb92470)

On further inspecting the Page and going to the Storage Section we found the cookie that might be useful for a later stage

![image](https://github.com/user-attachments/assets/a0444978-a586-4aa5-b74b-b6e5eec5c4fa)

" PHPSESSID Value : datlnccflgrmhqc9d5pkb6b8a7 "

We now know how to check the Cookie ID/SessionID Value ,
This process will be useful in the coming steps

Let’s try clicking on Forgot your Password?

![image](https://github.com/user-attachments/assets/4530f4ad-123d-4a2a-a12b-b432561b134b)

We are redirected to a new page → /reset_password.php , in order to reset the password we need to have an email id Let ‘s go back to the login page and Check the framework by checking the page source

( Ctrl + U )

![image](https://github.com/user-attachments/assets/d1e0eeda-53c6-4b5d-8abc-274b53a4b649)

Sweet ! we found that the Directory naming convention must be hmr_DIRECTORY_NAME

Time to Brute-Force the Directory by fuzzing

![image](https://github.com/user-attachments/assets/693994e8-516b-4c7a-8701-b0119b84b80e)

 ◈  scan command : 

◦ CMD : " ffuf -u 'http://hammer.thm:1337/hmr_FUZZ' -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -mc 200,301 "

![image](https://github.com/user-attachments/assets/e50d6c86-3ce8-4fe9-a731-10e143ccd38f)

◈ Let’s check Logs

http://hammer.thm:1337/hmr_logs

![image](https://github.com/user-attachments/assets/1d36c788-5157-4ae6-b6d0-6bd78fc5b772)

We find error.logs file in /hmr_logs/

![image](https://github.com/user-attachments/assets/ac3d11e4-5128-460a-8f12-f8d3e0ba0d00)

" email id found : tester@hammer.thm "

Let’s Head back to the login page @ port 1337 and try resetting the password using the email id we just found

![image](https://github.com/user-attachments/assets/d0adcc64-92bf-4812-ab57-9a6855de3f15)

After submitting the reset request for email id — tester@hammer.thm
we find that we are being redirected to an OTP based recovery code to in order to reset the password .

![image](https://github.com/user-attachments/assets/a9b619c8-f4cb-49ea-ba35-4953a3a9897f)

Since we need to reset the password we have to enter 4-Digit Recovery code , in order to attempt for all the 4 digit codes our maximum entries could be not more than 9999 for a 4 digit code and minimum can be 0000 also being a 4 digit code

We need to create a wordlist from 0000 to 9999
Here we will use the command seq ( Sequence command )
This will generate a wordlist file from 0000 to 9999 and would help us guess the recovery code for the password reset . Let’s name this wordlist as → codes

" seq 0000 9999 >> codes.txt "

![image](https://github.com/user-attachments/assets/3a1e4be8-bb2f-42b5-8d7c-9ee5ffa7d1f8)

Let’s now use the wordlist that we created above to guess the right 4 Digit recovery code by firing each one individually by Brute-forcing using the ffuf command

◈ Let’s prepare our payload first

Things we have to keep in mind :

1) As we saw @ the password reset page → we are only given 180 seconds to input the 4 digits recovery code , we need to override this limit because Brute-forcing 10000 digits can take a while to successfully find us the right combination for the 4-digit .
we will be using the command flag with ffuf → X-Forwarded-For
This will act as a Rate-Limit ByPass
Reference : HackTricks (Link)

Do Note : This can also be used with Burp Suite Method

2) We will be needing the cookie ( PHPSSESID ) that we found earlier

◈ Let’s Prepare our Payload

ffuf -w codes.txt -u “http://hammer.thm:1337/reset_password.php" -X “POST” -d “recovery_code=FUZZ&s=60” -H “Cookie: PHPSESSID=Cookie-ID” -H “X-Forwarded-For: FUZZ” -H “Content-Type: application/x-www-form-urlencoded” -fr “Invalid” -s

◈ Breakdown of the above command :

◦ -w codes.txt: Specifies the wordlist (codes.txt) that will be used for fuzzing. This wordlist contains the payloads that will replace the FUZZ keyword in the command.

◦ -u "http://hammer.thm:1337/reset_password.php": The URL of the target web application. This is where the fuzzing request will be sent.

◦ -X "POST": Specifies the HTTP method to be used, which in this case is POST.

◦ -d "recovery_code=FUZZ&s=60": The data being sent in the body of the POST request. The FUZZ keyword here will be replaced with each entry from codes.txt during the fuzzing process. It appears that the fuzzing is targeting the recovery_code parameter.

◦ -H "Cookie: PHPSESSID=datlnccflgrmhqc9d5pkb6b8a7": Adds a custom header to the request, specifically a Cookie header with a session ID. This is likely needed to maintain a session with the web application.

◦ -H "X-Forwarded-For: FUZZ": Adds another custom header, X-Forwarded-For, which is often used to identify the originating IP address of a client connecting to a web server. In this case, it's being fuzzed to see if the application behaves differently based on the IP address.
 
◦ -H "Content-Type: application/x-www-form-urlencoded": Specifies the content type of the data being sent. This is typical for form submissions.

◦ -fr "Invalid": Filters out responses that contain the string "Invalid". This helps in identifying successful or interesting responses that differ from the common invalid ones.

◦ -s: Runs ffuf in silent mode, which reduces the amount of output to only essential information.

◈ Let’s Fire it up 

We know that we have 180 seconds to input the new 4 digit recovery key .

lets prepare our payload and leave space for the cookie header section .

Then head back to the password reset page and quickly grab the new cookie ID and paste it into our payload and then execute it .

We have to do this under 180 seconds other wise a new cookie ID will be generated . After successfully executing the payload we will get the 4 Digit recovery code to reset our password

For making it easier i have added snippets of video to this writeup for you to follow

Let’s prepare our payload leaving the space for cookie ID and then we go to the reset password page and get the new cookie ID value and use it in our payload to launch the brute force attack ( We have 180 seconds to do this )

![image](https://github.com/user-attachments/assets/ba90edc5-ebee-43b2-81e5-627c88575c02)

We got our recovery code by running the command with the new Cookie ID , when we enter the 4 digit code that we brute forced we get redirected to the reset your password page for setting a new password

![image](https://github.com/user-attachments/assets/45fd80d8-a82c-4d47-9384-aed69baf372b)

Do note : The Recovery code that i found , may not be similar to yours
You can either enter the password recovery code or refresh the page .

We can now reset our password to any thing we want to
Here we are going to set it to 1234

![image](https://github.com/user-attachments/assets/6cb6f907-9dc5-4399-9ede-a41a7a08b728)

After setting the password to 1234 , we can see that the page redirects us back to the login page but with /index.php this time .
Let’s login using the email id — tester@hammer.thm and password — 1234 that we just reset

![image](https://github.com/user-attachments/assets/971f6aa7-a0ea-4a25-9317-a39afaef9664)

Voila ! We can see that we can now successfully login to the user account for whom the password we have just reset .

Congratulations we found our first flag ! Well Done !

◈ Finding the Second Flag

By successfully login in we got our first flag

If we inspect the page thoroughly we can see there is a search box which says enter command .

Let’s try entering the command — ls

![image](https://github.com/user-attachments/assets/74182b32-2ca0-47aa-8d4b-728aae427afe)

![image](https://github.com/user-attachments/assets/f400a459-8a25-4c4f-96a2-aa3aafb6012c)

Once we use the command ls we can see the return entries

188ade1.key

composer.json

config.php

dashboard.php

execute_command.php

hmr_css

hmr_images

hmr_js

hmr_logs

index.php

logout.php

reset_password.php

vendor

The first entry seems kind of interesting = 188ade1.key
Let’s try to read the contents of this file by using the command cat

![image](https://github.com/user-attachments/assets/db4845cb-37a9-4c8d-91ce-70c0774b87e1)

Whoops , the command cat is disabled and the output says —
command not allowed

We somehow have to find a way to check the contents of the file — 188ade1.key

Let’s check other options

Let’s try to explore using the page source — Ctrl+U |

![image](https://github.com/user-attachments/assets/e0da2131-db51-4f54-bbef-d864a0c54503)

![image](https://github.com/user-attachments/assets/2868d901-f668-46c6-9781-07f97fe4f3bc)

![image](https://github.com/user-attachments/assets/90041b73-5754-4408-9942-15529f956005)

There are a lot of results , but we will go with the first one

Let’s paste the jwt Token and check the results

![image](https://github.com/user-attachments/assets/5683bf56-890f-42e8-8145-4155b18fb133)

There is a need to input a 256 bit secret code for signature verification

![image](https://github.com/user-attachments/assets/dfb4586c-db48-4cc9-a192-8532ec6ca585)

Let’s use the 256 bit secret key we found inside the file → 188ade1.key

We find one interesting clue while trying to decode the jwt Token →

![image](https://github.com/user-attachments/assets/a88eaa99-0ffe-410b-91af-3714cef5991e)

By decoding the Token we find that the key location is →
/var/www/mykey.key

earlier we have found a key name — 188ade1.key by using ls command .

Which means this key is uploaded in /var/www/html →

Since the key is saved on the /var/www/html , we can directly open the file with the filename on the index page using it as a directory and down load it to our VM —

![image](https://github.com/user-attachments/assets/fcb19f3a-a272-4ce2-a815-bf97e3f8e09d)

Let’s open the file 188ade1.key to check the contents , we can’t open this file directly on Kali or Linux . To view the contents we have to install Office Libre ( This comes pre-installed in some Linux-Based-Distro )

To install Libre Office use the same command →

◦ CMD: "sudo apt install libreoffice libreoffice-gtk4"

On Opening the file we found the key →

"56058354efb3daa97ebab00fabd7a7d7"

Now let’s use this key as the 256 bit secret key

After entering the secret key we see that the signature gets verified

![image](https://github.com/user-attachments/assets/5cfa6801-f9ec-4137-81ea-63bee0e0c3f9)

To access sensitive data like a flag , we always have to have high privileges , the role is set to user , so Let’s try changing the role from user to admin .

( Verifying the signature is needed in order to manipulate the jwt Token )

![image](https://github.com/user-attachments/assets/fab63078-a350-42ed-90fe-768dc9ed0fa3)

After changing the role from user to admin , the token get manipulated .

"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L2h0bWwvMTg4YWRlMS5rZXkifQ.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzI1MzE0MzAzLCJleHAiOjE3MjUzMTc5MDMsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.YKBqC22iD4G-jXjjD9Y5j6mLjb62ZhR9b-un-8waUS4"

Since we have successfully changed our user to admin , now we can directly use the token to login as admin

Login back into the dashboard then Using Burp Suite to intercept the command and replacing the JWT Token as admin and then using cat /home/ubuntu/flag.txt →

![image](https://github.com/user-attachments/assets/a6a5c5b8-3a93-4134-81f4-2d91ecbe5e92)

**We get our Flag for the second Task**

![image](https://github.com/user-attachments/assets/00ee065f-634e-4364-8ff6-e6b385dee29a)

Room Done !

Flags

![image](https://github.com/user-attachments/assets/bfb390e9-13ff-44d9-a24e-a96ddb281bc4)

**THANK YOU ALL**









































