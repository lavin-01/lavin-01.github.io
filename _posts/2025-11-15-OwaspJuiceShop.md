---
title: "OWASP JUICE SHOP"
date: 2025-11-15 10:22:00 +0300
description: OWASP JUICE SHOP
image: /assets/images/OWASP/OWASP-Cover.png
categories: [WEB, Cryptography, Easy]
tags: [WEB, Cryptography, Easy, KenyaCyberlympics]
---
# OWASP Juice Shop Write-up

## Task 1: Introduction

The OWASP Juice Shop room provides a guided, interactive environment for investigating vulnerable online applications. Through a methodology that mimics contemporary penetration testing techniques, we get to engage directly with Juice Shop's vulnerabilities, which range from injection problems and faulty authentication to vulnerable setups. This room offers an approachable yet thorough environment for developing and honing web application security abilities.

### Set Up:

1. **Deploy the Attack box** or use **OpenVPN** to connect to the same network the machine is on
2. **Start the machine** in the first task and wait for a few minutes for it to load
3. **Confirm connectivity** by pinging the IP address generated to ensure you are connected to the same network as the machine

![image.png](/assets/images/OWASP/Connection.png)

## Task 2: Explore the Web Application

First, we need to understand the web application and its features. By clicking on the products and checking the reviews we obtain some emails:

- **admin@juice-sh.op**
- **jim@juice-sh.op**
- **bender@juice-sh.op**

### Q1: What is the administrator's email address?

**Answer:** `admin@juice-sh.op`

![image.png](/assets/images/OWASP/admin-email.png)

### Q 2: What parameter is used for searching?
**Answer:** `The parameter used for searching is q`
![image.png](/assets/images/OWASP/search-parameter.png)

### Q3: What show does Jim reference in his review?

Jim reviews a green smoothie made from green cabbage, spinach, kiwi and grass. He talks about a replicator which when searched from any web browser, we find that its a technology from a tv show called Star Trek

**Answer:** Jim is referring to the TV show **Star Trek** 
(specifically mentioning a "replicator" which is technology from the Star Trek universe).

![image.png](/assets/images/OWASP/wikipedia.png)

TASK 3: INJECTION ATTACKS
Injection vulnerabilities let attackers feed malicious input into an application to cause crashes, steal or alter data, or run unauthorized actions, making them high-risk for downtime and data loss. They're often easy to spot because bad input triggers errors. Common types include:
a) SQL injection -malicious DB queries to read/modify data or hijack accounts
b) Command injection - user input executed as system commands, e.g., abused ping utilities
c) Email injection -tampering form fields to send unauthorized emails

Question #1: Log into the administrator account!
When we use Burp suite proxy when the intercept is on and try logging in with the administrator email we found in the first steps, we get to see the request details.

![image.png](/assets/images/OWASP/admin-burp-result.png)

When we use SQL injection, specifically the Boolean based tautology injection, we find that the request is successful. This is because it forces the SQL query to always be true, then comments the rest of the statement.
In a poorly designed system, the first username is always the admin. Beginning with a letter “a” as in a’ or 1=1--, queries usernames beginning matching with that. This means that we are logging in as whoever happens to be the first user in the database which is almost always the admin. 

![image.png](/assets/images/OWASP/admin-burp-result-2.png)

Question #2: Log into the Bender account!
We found a few emails while exploring the website, one of them which was Bender’s  email address. So, we can use the tautology injection immediately after the email address and comment out the rest of the statement. This log in is successful because the tautology injection forces the SQL statement to always be true.

![image.png](/assets/images/OWASP/bender-burp-result.png)

![image.png](/assets/images/OWASP/bender-flag.png)


TASK 4: WHO BROKE MY LOCK
Question #1: Brute force the Administrator account's password!
Here, we use Burp suite proxy with the intercept on to log in using the admin email address. We then forward the request to intruder. Clear the password, then add comment equivalent in Burp. When then load our best1050.txt file which includes about 1050 known password combinations. We are going to filter the results using the status code, fishing for code 200 which means success. We find that the password is admin123 


![image.png](/assets/images/OWASP/admin-bruteforce.png)

![image.png](/assets/images/OWASP/code-200.png)


Question #2: Reset Jim's password!
We have Jim’s emails address, but we don’t have his password. The login page has a forgot password section which, when we click and use his email, we get a question asking for his brother’s middle name and then it allows us to change the password.
Using OSINT, we find that Jim has his family members listed on a Wikipedia page. He has one brother called George Samuel Kirk. We find the middle name “Samuel” to be able to proceed and reset his password.


![image.png](/assets/images/OWASP/family-wiki.png)

![image.png](/assets/images/OWASP/jim-flag.png)


TASK 5: SENSITIVE DATA EXPOSURE
Question #1: Access the Confidential Document!
When we go to the About Us page and hover the mouse over the highlighted Terms of Use we get the address to the link is 10.10.246.46/ftp/legal.md. In the parent directory, 10.10.246.46/ftp/ we find several other files that we can download and get sensitive information such as the backup files. 

![image.png](/assets/images/OWASP/about-us.png)
![image.png](/assets/images/OWASP/ftp-page.png)
![image.png](/assets/images/OWASP/confidential-doc-flag.png)


Question #2: Log into MC SafeSearch's account!
The account owner is a rapper with a YouTube video of his rap song. After watching the video there are certain parts of the song that stand out.
He notes that his password is "Mr. Noodles" but he has replaced some "vowels into zeros", meaning that he just replaced the o's into 0's.
We now know the password to the mc.safesearch@juice-sh.op account is "Mr. N00dles"

![image.png](/assets/images/OWASP/safesearch-acc.png)

Question #3: Download the Backup file!
The error message says that only .md and .pdf files are allowed. If we use the poison null byte which looks like %00 and add a .md file at the end we can be able to bypass the 403 error and download the back up file. Here we used %2500.md because the %00 raw null byte is usually sanitised or blocked, %2500 is URL encoded twice.


![image.png](/assets/images/OWASP/baxkup-file.png)

![image.png](/assets/images/OWASP/poison-null.png)

![image.png](/assets/images/OWASP/poison-null-flag.png)


TASK 6: BROKEN ACCESS CONTROL
a)	Horizontal Privilege Escalation
Occurs when a user can perform an action or access data of another user with the same level of permissions.
b)	Vertical Privilege Escalation
Occurs when a user can perform an action or access data of another user with a higher level of permissions.
Question #1: Access the administration page!
For this task, log in using the admin email address then right click the web page and click inspect. Navigate to Debugger and find the file named main-es2015.js. To get this into a format we can read, click the { } button at the bottom and search for the term “admin”

There are several results containing the word “admin” but we are specifically looking for "path: administration". This hints towards a page called "/#/administration" 


![image.png](/assets/images/OWASP/administration-page.png)

![image.png](/assets/images/OWASP/admin-inspect-result-2.png)

![image.png](/assets/images/OWASP/admin-flag.png)

Question #2: View another user's shopping basket!
Still in the Admin account, click on 'Your Basket'. Make sure Burp is running, and intercept is on so you can capture the request!
Forward each request until you see: GET /rest/basket/1 HTTP/1.1. change the number 1 after /basket/ to 2. It will now show you the basket of UserID 2. We can do this for other UserIDs as well.


![image.png](/assets/images/OWASP/another-user-basket-burp.png)

![image.png](/assets/images/OWASP/another-user-basket-2.png)

Question #3: Remove all 5-star reviews!
Navigate to the  http://MACHINE_IP/#/administration page again and click the bin icon next to the review with 5 stars.

![image.png](/assets/images/OWASP/delete-5star.png)

![image.png](/assets/images/OWASP/admin-flag-2.png)


TASK 7: CROSS-SITE SCIPTING
XSS or Cross-site scripting is a vulnerability that allows attackers to run javascript in web applications. These are one of the most found bugs in web applications. Their complexity ranges from easy to extremely hard, as each web application parses the queries in a different way. 
There are three major types of XSS attacks:
1.	DOM (Special)- DOM XSS (Document Object Model-based Cross-site Scripting) uses the HTML environment to execute malicious JavaScript. This type of attack commonly uses the <script></script> HTML tag.

2.	Persistent (Server-side)- Persistent XSS is JavaScript that is run when the server loads the page containing it. These can occur when the server does not sanitise the user data when it is uploaded to a page. These are commonly found on blog posts. 


3.	Reflected (Client-side)- Reflected XSS is JavaScript that is run on the client-side end of the web application. These are most commonly found when the server doesn't sanitise search data. 






Question #1: Perform a DOM XSS!
We will be using the iframe element with a JavaScript alert tag: 
<iframe src="javascript:alert(`xss`)"> 
Inputting this into the search bar will trigger the alert.


![image.png](/assets/images/OWASP/DOM-XSS.png)

![image.png](/assets/images/OWASP/DOM-flag.png)

Question #2: Perform a persistent XSS!
First, we login to the admin account.
We are going to navigate to the "Last Login IP" page for this attack. It should say the last IP Address is 0.0.0.0 or 10.x.x.x 
As it logs the 'last' login IP we will now logout so that it logs the 'new' IP. We have to make sure that burp suite is running and the intercept is on. We will then head over to the Headers tab where we will add a new header:

True-Client-IP	<iframe src="javascript:alert(`xss`)">

Then forward the request to the server!
When signing back into the admin account and navigating to the Last Login IP page again, we will see the XSS alert!

![image.png](/assets/images/OWASP/last-login-ip.png)

![image.png](/assets/images/OWASP/persistent-xss-burp-suite.png)

![image.png](/assets/images/OWASP/persistent-xss-alert.png)

Question #3: Perform a reflected XSS!
From there you will see a "Truck" icon, clicking on that will bring you to the track result page. You will also see that there is an id paired with the order.    
We will use the iframe XSS, <iframe src="javascript:alert(`xss`)">, in the place of the 5267-f73dcd000abcc353
After submitting the URL, refresh the page and you will then get an alert saying XSS!
The server will have a lookup table or database (depending on the type of server) for each tracking ID. As the 'id' parameter is not sanitised before it is sent to the server, we are able to perform an XSS attack.  


![image.png](/assets/images/OWASP/reflected-xss-alert.png)


TASK 8: SCOREBOARD
 O find the score board, we just need to navigate to the /#/score-board/ section on Juice-shop.

![image.png](/assets/images/OWASP/scoreboard.png)

The OWASP Juice Shop room on TryHackMe has provided me a solid, hands-on way to understand how real web vulnerabilities work and why insecure coding practices are so risky. By exploiting flaws like SQL injection, authentication bypasses, and file upload vulnerabilities, it has become clear to me how attackers think, and how small mistakes can turn into major security gaps. Working through these challenges has helped me build both offensive awareness and defensive intuition, ultimately strengthening my skills in identifying, exploiting, and preventing common web application threats.





