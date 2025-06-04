---
title: "Why Password Security Still Matters"
date: 2024-03-20
draft: false
summary: "An overview of why robust password practices—such as hashing, salting, and strong policies—remain essential defenses against both offline and online attacks."
---

![Passwords matter header image](/images/passwords-matter/header.jpg)

# Why Password Security Still Matters

Even in an era increasingly dominated by multi-factor authentication (MFA), passwords remain the primary gatekeeper for countless systems and services. Data breaches consistently reveal that weak or compromised passwords are among the top culprits for unauthorized access. For example, public breaches often expose password hashes which attackers can crack offline, granting them immediate entry to user accounts. As long as passwords are used, understanding how they are stored, why hashing and salting are essential, and how attackers attempt to reverse them is crucial for both security practitioners and end users.

# Authentication Factors & Access Control Basics

Effective access control hinges on four interconnected components: *Identification, Authentication, Authorization, and Accountability.*

**Identification** asks, “Who are you?” and typically involves presenting a username or user ID.

**Authentication** verifies that the presented identity is genuine, most commonly via passwords (knowledge factors), but also via ownership factors (like hardware tokens) or characteristic factors (biometrics).

**Authorization** determines what the authenticated user is allowed to do—what resources they can access and what actions they can perform.

**Accountability** tracks and logs user actions, ensuring that any changes or accesses can be traced back to a specific identity.

When designing secure systems, it’s important to distinguish between policy definition (establishing “who should have what access”) and policy enforcement (the mechanisms, software or hardware, that enforce those rules). For example, a policy may state that only members of the “Admins” group can modify server configurations, while enforcement is handled by an access control list (ACL) on the server or centralized directory service.

Authentication factors are traditionally grouped into three categories:

- Something you know: Passwords, PINs, or passphrases.

- Something you have: Hardware tokens (like YubiKeys), software tokens (such as TOTP apps), or smart cards.

- Something you are: Biometrics, including fingerprints, facial recognition, or iris scans.

While MFA (e.g., combining a password with a one-time code from a hardware token) significantly increases security, the initial “something you know” factor still relies on proper password storage and handling to remain effective.

# Password Storage 101

Storing passwords in plaintext is universally recognized as a critical security failure: if an attacker acquires the password database, every account is immediately compromised. To avoid this, modern systems use cryptographic hash functions to transform a plaintext password into a fixed-size string of characters that cannot feasibly be reversed.

A cryptographic hash function can include some important properties:

**Determinism:** The same input always produces the same output.

**Fixed-size output:** Regardless of input length, the hash is of a fixed length (e.g., 256 bits for SHA-256).

**Preimage resistance:** Given a hash value, it should be computationally infeasible to find any input that maps to that hash.

**Collision resistance:** It should be hard to find two distinct inputs that produce the same hash.

Common hash algorithms used in practice include the SHA (Secure Hash Algorithm) families—SHA-1, SHA-256, SHA-512—and newer standards like SHA-3. Non-cryptographic hashes (e.g., CRC32) are fast but lack the collision resistance needed for secure password storage.

A typical password storage workflow looks like this: *when a user sets a password, the system computes hash = H(password) and stores hash (not the plaintext). When the user logs in, they provide their password again, the system computes H(provided_password), and compares it to the stored hash. If they match, authentication succeeds.* However, this alone is insufficient against offline attacks, where an attacker steals the password database containing these hashes and attempts to reverse them through brute-force or dictionary attacks.

# Salting and Slow Hashing

To mitigate offline attacks, two primary techniques are used: salting and slow hashing.
What Is a Salt?

A salt is a random string, typically unique per user, that is concatenated with the password before hashing. Formally, if s is the salt and p is the password, the stored value is H(s ∥ p) where “∥” denotes concatenation. The salt is stored in plaintext alongside the hash (e.g., in the /etc/shadow file on Linux systems). By including a unique salt per password, two users with the same password will have different hash outputs, and precomputed rainbow tables become useless unless they were generated for that specific salt.

For example, on many Unix-like systems, the /etc/shadow entry for a user looks like:

```bash
alice:$6$YKGPfU93QZBdWd38$kRmiZ5e1B48kdQxVhkZ6BvY...F5:...
```

Here,

    $6$ indicates SHA-512,

    YKGPfU93QZBdWd38 is the salt,

    the long string after the salt is the SHA-512 hash of salt ∥ password.

## Example of extracting a salt value

As a test, we create a user with the login/password values goodguy/password. Reading the shadow-file, we see the entry for the newly added user, indicated below with a green square. We should see something like:

goodguy:$y$j9T$Pr/BoHmQ8ZjNTwj2VRU08.$0ltkT9US3Pe4C6pd202gWEDknIdTy8aPtrcg6snf0b9:19766:0:999:7:::

In this field, the first dollar‐sign group ($y$) indicates the hashing algorithm/parameters, the next group (j9T) is the internal cost factor, and the third group (Pr/BoHmQ8ZjNTwj2VRU08.) is the actual salt. When we combine them as "$y$j9T$Pr/BoHmQ8ZjNTwj2VRU08.", that entire string becomes what Python needs to re‐compute the hash.

![Creating user and checking the shadow file](/images/passwords-matter/1.jpg)

Next, use Python’s built‐in crypt library to verify the hash. As we know the goodguy’s plaintext password is literally *password*.

```bash
python3 - << 'EOF'
import crypt

salt_field = '$y$j9T$Pr/BoHmQ8ZjNTwj2VRU08.'
hashed = crypt.crypt('password', salt_field)
print(hashed)
EOF
```

![Extracting the salt](/images/passwords-matter/2.jpg)

That matches exactly the string we saw in /etc/shadow for goodguy, demonstrating that we've correctly extracted the salt and re‐generated the same hash. In other words, we’ve proven that the salt field (Pr/BoHmQ8ZjNTwj2VRU08.) is all we need—along with the algorithm identifier—to recreate or verify goodguy’s password hash.

## Reversing MD5 Hashes

Despite MD5’s known cryptographic weaknesses, many legacy systems still use it for non-critical checksums, and sometimes (unwisely) for password hashing. Since MD5 hashes are fast to compute and have been extensively precomputed, reversing them via online lookup or rainbow tables is trivial.

As a test, I have previously been asked to e.g. inspect hashed passwords stored in databases, on web servers user overviews, and so on. Finding a list of usernames alongside their MD5-hashed passwords is not uncommon even these days.

**Example MD5 hash**: 13c3a11755a5ebec80f457cc5c105cbc.

Reverse Lookup - By submitting this MD5 hash (13c3a11755a5ebec80f457cc5c105cbc) to an online MD5 reverse-lookup tool, the plaintext password kissa123 was retrieved, and quickly at that!

<img src="/images/passwords-matter/3.jpg"
     alt="Reverse MD5 Lookup"
     width="300" />


# Offline vs. Online Attacks

When an attacker steals a database of unsalted or weakly protected hashes, they can perform offline attacks—attempting guesses at their leisure without interacting with the live authentication system. Two primary offline attack methods are:

## Dictionary Attacks

Using a wordlist (e.g., the “rockyou” wordlist), attackers hash each candidate password with the known salt (if any) and compare to the stolen hash.

If the password is common (e.g., “password123”), it is found quickly—in seconds or less, depending on the hash algorithm’s speed.

## Brute-Force Attacks

Trying all possible combinations of characters, starting from shortest to longest, until the correct password is found.

The time required grows exponentially with each additional character and character class (lowercase, uppercase, digits, symbols).

As a rule of thumb, a 5–6 character alphanumeric password can be cracked in seconds; a 7–8 character password in minutes to hours on a modern GPU-equipped cracking rig. Longer or more complex passwords become infeasible.

## Rainbow-Table Attacks

Precomputed tables of hashes for every possible password up to a certain length and character set.

Without salts, an attacker loads a rainbow table into memory and instantly looks up a hash to find its corresponding password.

With a unique salt per password, a new rainbow table is needed for each salt, rendering rainbow tables largely ineffective against properly salted hashes.

By contrast, online attacks involve repeatedly submitting guesses to the authentication service itself. Though slower (since network latency and server response times come into play), they can target rate-limited systems. Good defenses against online attacks include account lockouts, CAPTCHA challenges, and MFA.

## Hands-On: Cracking a Real Password

To illustrate how these attacks work in practice, let’s walk through an exercise I previously completed, in which a password-protected ZIP archive was cracked, and then a Linux shadow file was bruteforced. We are finding the password for the zipfile TryCrackMe.zip on the kali VM desktop. Within the file we have a textfile containing a token.

<img src="/images/passwords-matter/4.jpg"
     alt="zip file"
     width="400" />

By using the fcrackzip tool (a utility designed for ZIP password cracking) and supplying the widely used “rockyou” wordlist (/usr/share/wordlists/rockyou.txt in most kali distros), the correct password was discovered.

<img src="/images/passwords-matter/5.jpg"
     alt="using fcrackzip"
     width="400" />

## Hands-On: Cracking a linux shadow file

As a test, we use grep to search the shadow file (/etc/shadow) for the entry for the user called *victim*. By copying that to a separate file named *shadow*, we stored the entry (the entire line). Looking at the hash, we see the algorithm parameter is listed as $6$, which would be SHA-512.

<img src="/images/passwords-matter/6.jpg"
     alt="shadow file"
     width="600" />

Continuing, we do the same for the passwd file entry (/etc/passwd), redirecting the output to a new file called passwd (instead of manually copying using nano, as done for the shadow part).

<img src="/images/passwords-matter/7.jpg"
     alt="passwd file"
     width="600" />

The unshadow command merges our copies of Linux's /etc/passwd and /etc/shadow files to produce a single file containing both user account details and hashed passwords in a format suitable for cracking. 

<img src="/images/passwords-matter/8.jpg"
     alt="using unshadow"
     width="600" />

With John the Ripper, we can then use this combined file, applying its specified hash algorithm (e.g., SHA-512) and wordlists, to systematically attempt to match hash values against potential passwords. When a hash match is found, John outputs the plaintext password, allowing verification or unauthorized access detection.

<img src="/images/passwords-matter/9.jpg"
     alt="johntheripper formats"
     width="600" />

As I ran several tests and tried different parameters, I made copies of the victim.txt and in the last successful file it was named shadowcopy.txt.

<img src="/images/passwords-matter/10.jpg"
     alt="running john using rockyou.txt and the shadow and passwd file combined"
     width="600" />

Testing the password *chucknorrisrules80* to log into the victim user in my Kali box is successful. 

<img src="/images/passwords-matter/11.jpg"
     alt="successful login using the cracked password"
     width="600" />


# Protecting Against These Attacks

Given the variety of offline and online cracking techniques, defense-in-depth is essential. As a quick summary:

## Enforce Strong, Unique Passwords

Length and complexity: Require at least 12 characters, mixing uppercase, lowercase, digits, and symbols. Longer passphrases (e.g., a sentence or four random words) can be both memorable and secure.

Password managers: Encourage users to adopt password managers (e.g., Bitwarden, LastPass) over reusing or weak passwords.

## Use Salted Slow Hash Algorithms

Avoid fast hashes like MD5, SHA-1, or raw SHA-256 for password storage. Instead, use bcrypt, Argon2, or PBKDF2 with a high iteration count.

Ensure each user has a unique, cryptographically random salt at least 16 bytes in length.

## Rate Limiting and Account Lockouts

Implement throttling (e.g., no more than five failed login attempts per hour) to frustrate online dictionary or brute-force attempts.

Employ temporary account lockouts or CAPTCHA challenges after multiple failures.

## Multi-Factor Authentication (MFA)

Whenever possible, add a second factor (e.g., TOTP app, hardware token, SMS code) to complement passwords. Even if an attacker obtains a password, they cannot log in without the second factor.

Be mindful of MFA pitfalls (e.g., SMS-based codes can be intercepted via SIM swapping), and prefer authenticator apps or hardware keys.

## Monitor and Respond

Regularly audit systems for weak or reused passwords. Tools like “Password Spraying Detector” can identify anomalous authentication patterns.

If a breach is detected, force password resets for exposed accounts.

## Educate Users

Teach users (employees, friends, family, whoever!) about phishing, social engineering, and the importance of never reusing passwords across multiple sites.

Encourage periodic password changes if there’s evidence of exposure (though forced periodic rotation without cause can lead to weaker passwords or insecure storage).

By combining these measures, strong password policies, proper hashing and salting, MFA, and vigilant monitoring, we can drastically reduce the risk posed by offline cracking and online guessing attacks.

# Conclusion & Further Reading

I hope this rather comprehensive guide has helped and walked through the theory behind password security—covering access control fundamentals, cryptographic hashes, salting, and slow hashing, as well as practical demonstrations of how attackers reverse insecure password stores. By understanding and implementing these best practices, security teams and others alike can strengthen their security!

For more in-depth reading, make sure to check out:

- NIST SP 800-63B: Digital Identity Guidelines on Authentication and Lifecycle Management.

- OWASP Password Storage Cheat Sheet: Recommendations on secure password hashing and storage.

- CyberChef (GCHQ’s “Cyber Swiss Army Knife”): A flexible web-based tool for encoding, decoding, and hashing data.

- John the Ripper & Hashcat Documentation: For learning advanced configuration and GPU-based cracking techniques.

Maintaining secure passwords is an ongoing process: as computational power grows and new attack techniques emerge, organizations (and all of us) must continually reevaluate hashing parameters, policy enforcement, and user education to stay ahead of adversaries.