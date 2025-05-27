---
title: "Data Networks Episode 13: Configuring Apache for Distributing Resources"
date: 2023-04-02
draft: false
summary: "Setup and testing of the Apache web server for internal network resource distribution. Includes configuring permissions, analyzing HTTP traffic using Wireshark, and updating router/switch configuration files."
---

## Plan

- [ ] Enable & Test Apache Webserver
- [ ] Upload web-portfolio as the hosted page
- [ ] Wireshark testing
- [ ] Update configuration files with E13 changes

# Verify lubuntu-apache instance

Using Lubuntu1, checking the webserver service and downloading the index-file found on the localhost address:

>![](/images/network-doc/E13/testinglocalhost.gif)

Adding a new website as the webserver page:

>![](/images/network-doc/E13/copywebtech.png)

Changing permissions so anyone can access and read the files. I do a test from Lubuntu3, connecting to the ip of Lubuntu1 http://192.168.39.5/

>![](/images/network-doc/E13/testingportfolio.gif)

- [x] Enable & Test Apache Webserver
- [x] Upload web-portfolio as the hosted page

# Sniffing the traffic to and from the webserver

As shown in the picture below, I add a lubuntu with wireshark in place:

>![](/images/network-doc/E13/wiresharklocation.png)

Continuing, I boot up the wireshark and start capturing packets. As it is capturing, I open the browser on Lubuntu3 and insert the address to Lubuntu1, as before.

As seen in the **GIF** below, we see the result as several packages between .5 and .135. 

>![](/images/network-doc/E13/capturingpackets.gif)


While the client requests with GET, and we can see information about the source, host and also the user-agent. The user-agent simply describes which software or user information the website might want/need to adapt the content to best suit that type of user. Having an OSINT-background, I could not resist changing the user-agent as that is such a simple procedure. As we see int he gif, the new agent shows immidiately. 

>![](/images/network-doc/E13/GET.png)

While the server responds with the content, as seen in the picture below. Initially in the subject-line it is described as HTTP/1.1, which would describe the HTTP version. The source IP is the Lubuntu1, sending the requested information through port 80
 as we know are used for HTTP. Furthermore the TCP *view* shows the type of server "Apache/2.4.41" running on an Ubuntu based distro. 

>![](/images/network-doc/E13/http.png)


## Deeper dive into the packets

If we search for the stream 0 that I find in the information about the first HTTP packets, we get information about this communication. Initially we see the three-way-handhake syn/ack-syn/ack that we looked into in the previous exercise. 

>![](/images/network-doc/E13/3wayhandshakegetandok.png)

As described above, we know that HTTP uses port 80 and TCP as transport layer protocol. As seen in the screenshot from the above wireshark capture, we see the 3-way-handshake, then a GET request from .135 to .5 to get access to the file mainstyles.css.
further down the list, we see that the HTTP OK is sendt.

- [x] Wireshark testing


## HTTP GET


>![](/images/network-doc/E13/Get_deep.png)

|#|Description|
|---|---|
|1.| Request method: GET|
|2.| Request URI: mainstyles.css|
|3.| Request version: HTTP 1.1|
|4.| The host Lubuntu1|
|5.| User-agent: The client (.135 is showing that they are using mozilla to read the page)|
|6.| Accept: text/css, tells the server what kind of file it can expect. The client is expecting a text-file/css file|
|7.| Accept-language: accepting language standard|
|8.| Accept-Encoding: Accepting encoding on the client side|
|9.| After the current transaction finishes, this controls weather the network connection stays open. The type here is keep alive. |


## HTTP OK

>![](/images/network-doc/E13/HTTPOK.png)

|#|Description|
|---|---|
|1.| Response version and codes: Response version HTTP 1/1 and status code 200 sent by the server, as well as response phrase OK also sent by the server. The HTTO GET has succeeded. |
|2.| Date: Current date/time in GMT when the HTTP GET was received by the server|
|3.| Server: The server details|
|4.| Last-modified: When the mainstyles.css file was last modified.|
|5.| ETag: From a google-search it stands for Entity tag, which is an identifier for a specific version of a resource. It lets caches be more efficient and save bandwidth, as a web server does not need to resend a full response if the content was not changed. |
|6.| Accepted-Ranges: Bytes. The unit used in the server for content is bytes. |
|7.| Content-lenght: the total length of the file mainstyles.css|
|8.| Keep-alive: parameters for the keep-alive connection status, weather it stays open after the current transaction. |
|9.| Content-type: the content of the file mainstyles.css is text/css. |
|Bottom|The actual content of the textfile can be read.|


## Updated configuration-files after completing E13

[Network_Switch_A](/images/network-doc/E13/Config_files/E13-SwitchAu.cfg)\
[Network_Switch_B](/images/network-doc/E13/Config_files/E13-SwitchBu.cfg)\
[Network_Switch_C](/images/network-doc/E13/Config_files/E13-SwitchCu.cfg)

[Vyos_Router_A](/images/network-doc/E13/Config_files/E13-RouterAu.cfg) \
[Vyos_Router_B](/images/network-doc/E13/Config_files/E13-RouterBu.cfg) \
[Vyos_Router_C](/images/network-doc/E13/Config_files/E13-RouterCu.cfg) 

- [x] Update configuration files with E13 changes
