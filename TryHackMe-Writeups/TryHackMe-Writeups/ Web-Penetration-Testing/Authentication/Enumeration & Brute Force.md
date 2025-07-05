**TryHackMe | Enumeration & Bruteforce**

⦿ After studying the concept of Reconnaissance, I’ve come to understand the critical role that information gathering plays in the early stages of a penetration test. Reconnaissance is typically divided into two main categories: Passive and Active, both of which serve as the foundation for identifying potential entry points and understanding the target surface.

In this task, I begin exploring the topic of Enumeration and Brute Force Attacks, specifically in the context of web authentication mechanisms.

Authentication Enumeration is a core aspect of web application security testing. It involves analyzing how an application handles authentication-related functionality, including username validation, password policies, error messages, and session management. Improper implementation of these mechanisms can reveal sensitive information, making the application susceptible to further attacks such as brute force or credential stuffing.

◎ TASK 1: START THE MACHINE

The first step in this exercise is to start the virtual machine provided by TryHackMe. Once the machine is fully deployed and an IP address is assigned, I need to add the IP to my local /etc/hosts file for name resolution. The entry should be added in the following format:

![image](https://github.com/user-attachments/assets/34c4529a-caf9-4796-a231-efdc4b037191)

next is to visit http://enum.thm to access the machine.

◎ TASK 2: AUTHENTICATION ENUMERATION

Authentication Enumeration is the process of discovering valid usernames or user accounts by analyzing how an application handles authentication attempts. This typically involves examining system responses to different login attempts to infer whether a username exists or not — a common weakness in many web applications.

Attackers often exploit authentication mechanisms on platforms such as Gmail, Facebook, or other social services by attempting to log in with various usernames and observing the responses. If the application returns distinct messages like “This account doesn’t exist” vs “Incorrect password”, it reveals valuable information. For example, receiving an “Incorrect password” message typically implies that the username is valid, which allows attackers to shift their focus toward cracking the password.

◈ IDENTIFYING VALID USERNAMES

Applications may unintentionally leak information about valid usernames through:

◦ Login error messages

◦ Password reset forms

◦ Account recovery responses

By comparing these responses, attackers can determine whether a specific username exists in the system. If different error messages are displayed for valid and invalid usernames, this becomes a clear vector for authentication enumeration.

◈ PASSWORD POLICIES

Password policies define the structure and complexity requirements for user passwords, such as the inclusion of:

◦ Uppercase letters

◦ Special characters or symbols

◦ Minimum/maximum length

◦ Numbers

Understanding these policies allows an attacker to estimate the strength or pattern of expected passwords. For instance, if the application enforces strong password rules, brute-force attempts may need to account for more complex combinations.

Example: The following PHP code snippet uses a regular expression to enforce a strong password policy requiring symbols, numbers, and uppercase letters:

**php code**

"<?php
$password = $_POST['pass']; // Example1
$pattern = '/^(?=.*[A-Z])(?=.*\d)(?=.*[\W_]).+$/';

if (preg_match($pattern, $password)) {
    echo "Password is valid.";
} else {
    echo "Password is invalid. It must contain at least one uppercase letter, one number, and one symbol.";
}
?>"

If the supplied password doesn’t satisfy the policy defined in the pattern variable, the application will return an error message revealing the regex requirements. An attacker can use this information to generate a custom dictionary that matches the expected password pattern — increasing the effectiveness of their brute-force or credential-stuffing attacks.

◈ COMMON PLACES TO ENUMERATE

Web applications often include user-friendly features that, if not properly secured, can unintentionally leak sensitive authentication data. Here are several common areas where authentication enumeration is likely to occur:


◈ REGISTRATION PAGES

During the registration process, the application typically checks if a username is already taken. The system's response — whether the username is available or not — can help an attacker determine if an account already exists. For example:

◦ "Username already exists" → Account confirmed to exist

◦ "Username available" → Account likely does not exist

◈ PASSWORD RESET FEATURES

Password reset mechanisms ask users to submit an identifier (like a username or email) to initiate account recovery. If the application responds with different messages based on whether the input exists or not (e.g., "Account not found" vs "Check your inbox"), it enables attackers to confirm valid accounts.

◈ VERBOSE ERRORS

As mentioned earlier, verbose error messages are a key source of information leakage. These are detailed responses such as:

◦ "Invalid username"

◦ "Incorrect password"

Such messages help users troubleshoot login issues but also make it easier for attackers to distinguish between valid and invalid usernames — aiding in authentication enumeration.

☑ Answer to the question:
What type of error messages can unintentionally provide attackers with confirmation of valid usernames?

→ Verbose Errors

◎ TASK 3: ENUMERATING USERS VIA VERBOSE ERRORS

◈ UNDERSTANDING VERBOSE ERRORS

Verbose errors are unintentional leaks of sensitive information, often generated to assist developers during debugging. While they help diagnose issues, these messages can also provide attackers with valuable internal insights about an application’s structure and behavior.

○ Verbose errors can reveal:

 ◦ Internal Paths – Absolute file paths or directory structures, which may expose configuration files or secrets.

 ◦ Database Details – Information about database schemas, table names, or SQL syntax.

 ◦ User Information – Hints about the existence of specific users, emails, or usernames.

◈ INDUCING VERBOSE ERRORS

Attackers use several methods to deliberately trigger verbose error messages. These errors can be used to enumerate users, understand system logic, or identify misconfigurations.

◈ INVALID LOGIN ATTEMPTS

Submitting incorrect usernames or passwords allows an attacker to observe how the system responds:

◦ "Incorrect password" suggests that the username exists.

◦ "User does not exist" confirms the username is invalid.

◦ This difference is a clear sign of user enumeration.

◈ SQL INJECTION

By injecting special characters like a single quote (') into input fields, attackers may cause the backend to throw SQL errors. These responses can reveal:

◦ The type of database being used.

◦ Table or column names.

◦ Query structures.

☆ EXAMPLE:

"Input: ' OR 1=1 --

Response: SQL error near '1=1'
"

◈ FILE INCLUSION / PATH TRAVERSAL

By manipulating file paths in URL parameters (e.g., using ../../ sequences), attackers may trigger errors that disclose internal directory structures or system files.

☆EXAMPLE:

"GET /page.php?file=../../../../etc/passwd"

◈ FORM MANIPULATION

Tampering with form fields or hidden parameters can cause the application to return detailed validation errors. These messages may unintentionally reveal input formatting rules or backend logic.

◈ APPLICATION FUZZING

Sending unexpected or malformed input to the application in large volumes using automated tools (such as Burp Suite Intruder or ffuf) can help discover unhandled exceptions or weak validation mechanisms that result in verbose errors.

◈ THE ROLE OF ENUMERATION AND BRUTE FORCING

USER ENUMERATION

Identifying valid usernames reduces the scope of brute-force attacks by allowing the attacker to focus only on confirmed user accounts.

◈ EXPLOITING VERBOSE ERRORS

◦ Verbose errors can provide information on:

◦ Password complexity rules.

◦ Rate limiting or lockout mechanisms.

◦ Error handling workflows.

This intelligence can guide attackers in crafting more targeted brute-force attacks.

◈ ENUMERATION IN AUTHENTICATION FORMS

An example of this vulnerability can be found in real-world applications such as "Forgot Password" or login forms. A HackerOne report demonstrated how an attacker was able to enumerate users through the password recovery feature by observing error responses.

☆ LAB EXAMPLE:

Navigate to:

"http://enum.thm/labs/verbose_login/

Enter any email address into the "Email" input field."

Observed Behavior:

If the email is not registered, the site responds with:

◦ "Email does not exist"

This response confirms whether or not an email is registered in the system — a clear vulnerability resulting from verbose error messaging.

![image](https://github.com/user-attachments/assets/601232a0-6589-481c-b9e6-8784707081da)

However, if the email is already registered, the website will respond with an “Invalid Password” error message, indicating that the email exists in the database but the password is incorrect.

![image](https://github.com/user-attachments/assets/b2a41731-8f4c-43c5-9fa1-4f8353742a04)

◈ AUTOMATION

Below is a Python script that will check for valid emails in the target web app. Save the code below as script.py

"import requests
import sys

def check_email(email):
    url = 'http://enum.thm/labs/verbose_login/functions.php'  # Location of the login function
    headers = {
        'Host': 'enum.thm',
        'User-Agent': 'Mozilla/5.0 (X11; Linux aarch64; rv:102.0) Gecko/20100101 Firefox/102.0',
        'Accept': 'application/json, text/javascript, */*; q=0.01',
        'Accept-Language': 'en-US,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate',
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
        'X-Requested-With': 'XMLHttpRequest',
        'Origin': 'http://enum.thm',
        'Connection': 'close',
        'Referer': 'http://enum.thm/labs/verbose_login/',
    }
    data = {
        'username': email,
        'password': 'password',  # Use a random password as we are only checking the email
        'function': 'login'
    }

    response = requests.post(url, headers=headers, data=data)
    return response.json()

def enumerate_emails(email_file):
    valid_emails = []
    invalid_error = "Email does not exist"  # Error message for invalid emails

    with open(email_file, 'r') as file:
        emails = file.readlines()

    for email in emails:
        email = email.strip()  # Remove any leading/trailing whitespace
        if email:
            response_json = check_email(email)
            if response_json['status'] == 'error' and invalid_error in response_json['message']:
                print(f"[INVALID] {email}")
            else:
                print(f"[VALID] {email}")
                valid_emails.append(email)

    return valid_emails

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 script.py <email_list_file>")
        sys.exit(1)

    email_file = sys.argv[1]

    valid_emails = enumerate_emails(email_file)

    print("\nValid emails found:")
    for valid_email in valid_emails:
        print(valid_email)"

        


