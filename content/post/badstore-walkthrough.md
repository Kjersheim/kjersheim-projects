---
title: "BadStore.net Security Challenge: An Investigative Walkthrough"
date: 2024-01-10
draft: false
summary: "Exploring enumeration, SQL injection, privilege escalation, and web security weaknesses through a methodical analysis of the vulnerable BadStore.net web app."
---

This article documents the methodology used to solve a security challenge against the intentionally vulnerable web application **BadStore.net**. The walkthrough emphasizes the steps, tools, and techniques employed during the investigation, while omitting any real flags or sensitive credential values discovered along the way.

---

## Initial Enumeration and Manual Exploration

After accessing the main page, I reviewed the page source to identify accessible paths, script files, and other linked resources.

![Source page and file references](/images/badstore-images/1.jpg)

Navigating manually by altering the URL path (e.g. `/admin`, `/cgi-bin`) revealed a **Secret Administration Menu**, although access was restricted unless authenticated.

![Restricted admin menu](/images/badstore-images/2.jpg)

Examining the guestbook feature led to a discovery: clicking “Add Entry” exposed previously submitted posts, complete with **names and email addresses**.

![Guestbook entries exposed](/images/badstore-images/3.jpg)

This information became helpful during login and password reset testing, confirming that resetting passwords triggered backend changes even without authentication. However, logging in still failed, indicating either faulty logic or an additional protection layer.

---

## Port Scanning and Nikto

With enumeration complete, I moved on to active scanning. Using **Nmap**, I scanned `www.badstore.net` for open ports:

![Nmap results](/images/badstore-images/4.jpg)

Open ports included:

- **80 (HTTP)**
- **443 (HTTPS)**
- **3306 (MySQL)**
- **22 (SSH)**
- **3389 (RDP)**

These findings suggested available attack surfaces for both web-based and backend attacks.

A **Nikto scan** identified two interesting indexed directories:

- `/backup/`
- `/supplier/`

![Nikto indexed directories](/images/badstore-images/5.jpg)

Both exposed directory listings, which revealed files containing **user credentials** in a `username:password` format. These passwords appeared hashed or encoded.

---

## Exploiting SQL Injection

Accessing the login form and inputting the classic payload:

```
'or 1=1 or '
```

...allowed a **successful bypass** of authentication for a test user account.

![SQL Injection success](/images/badstore-images/6.jpg)

The SQL injection trick manipulated the underlying SQL logic:

```sql
SELECT * FROM users WHERE username='' OR 1=1 OR '' AND password='...'
```

This forces the query to always return true.

Attempts to use similar payloads (e.g. `admin'/*`) to gain access to administrator privileges initially failed. However, after restarting the Kali VM and clearing browser sessions, the injection worked again for the test user—but not for the admin account.

---

## Privilege Escalation via POST Injection

A more creative path was needed. I switched tactics and inspected the registration form source code.

I modified a local copy of the form’s HTML to set the `U` field (user role) to `A`, designating an admin account. Using the IP address of BadStore.net in the `action` attribute of the form ensured proper POST submission.

Opening the locally saved HTML file resulted in a working form page:

![Local form opened](/images/badstore-images/7.jpg)

After submitting the form with elevated privileges...

![New user form](/images/badstore-images/8.jpg)

...I was able to log in and access the **Secret Administration Menu**:

![Admin page access](/images/badstore-images/9.jpg)

Inside the admin panel, a list of users was visible:

![User list](/images/badstore-images/10.jpg)

...including the newly created administrator account.

---

## Hash Identification and Cracking

The admin user table revealed hashed passwords. To analyze them, I copied a hash and tested it against online MD5 databases.

![Testing hash online](/images/badstore-images/11.jpg)

One result confirmed the hash was created using MD5 and revealed the plaintext word.

To validate this locally, I generated a hash from the word "secret":

![Local MD5 testing](/images/badstore-images/12.jpg)

Using tools like `hashcat` or `john`, and the `rockyou.txt` wordlist, I cracked additional hashes:

![Hash cracking with rockyou.txt](/images/badstore-images/13.jpg)

This verified that weak passwords like “1234” were used by some accounts, including the one I created earlier.

---

## Brute-Force Attempts with Hydra

To test brute-force protection, I used Hydra on the login form:

```bash
hydra -l admin -P rockyou.txt www.badstore.net http-post-form "/cgi-bin/badstore.cgi?action=login:email=^USER^&passwd=^PASS^&Login=Login:Password not found" -V
```

---

## Web Exploits and Checkout Manipulation

Logged in as the administrator, I added items to the cart:

![Cart view](/images/badstore-images/14.jpg)

During checkout, inspecting the HTML revealed this function:

```html
onsubmit="return DoCardvrfy(this);"
```

I altered the line to bypass client-side credit card verification:

```html
onsubmit="return;"
```

This removed validation and let the transaction go through without a valid card:

![Checkout HTML modified](/images/badstore-images/15.jpg)

---

## Tampering with Burp Suite

Next, I used **Burp Suite** to intercept and manipulate form submissions.

I turned intercept on and submitted payment data:

![Burp intercept on](/images/badstore-images/16.jpg)

In the developer tools of Burp’s built-in browser, I edited the JavaScript card verification logic to always return `true`:

![Burp JS edited](/images/badstore-images/17.jpg)

I then modified the intercepted packet before sending it to the server:

![Packet editing](/images/badstore-images/18.jpg)

To further test tampering, I changed the price of an item:

![Price altered](/images/badstore-images/19.jpg)

The price was successfully altered from 5000 to 12.3.

---

## Final Thoughts

This segment of the BadStore.net challenge demonstrates how weak input validation, exposed backend logic, and insecure password practices can be chained together to compromise an entire system:

- SQL injection enabled login bypass.
- POST tampering allowed privilege escalation.
- Hashes were weak and reversible.
- Client-side validation was easily bypassed.
- Burp Suite offered full control of submission payloads.

These vulnerabilities illustrate why defense-in-depth and secure coding practices are important in modern web applications.

