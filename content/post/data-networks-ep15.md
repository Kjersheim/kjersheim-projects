---
title: "Data Networks Episode 15: Configuring Basic DNS Functionality"
date: 2023-04-16
draft: false
summary: "This entry explores DNS queries using `dig +trace`, packet capture via Wireshark, and building a local DNS list using the /etc/hosts file. Detailed breakdowns of DNS packet analysis and validation through SSH, HTTP, ping, and traceroute are included."
---

# Plan 

- [ ] Test and document dig towards a labranet address
	- [ ] Capture packets and analyze

- [ ] Create a local DNS list in /etc/hosts
	- [ ] Testing, verifying and documenting



# Capturing DNS Data 

From the previously discussed topologies I want to do some testing from one VM to the internet. On the Lubuntu1 I change its interface from internal network to a bridge to the hostmachine.
This so it gets access to the internet isntead of the virtual network I have created. I proceed to start up Wireshark on the VM.

To avoid getting an abundance of information, I do some reconnaissance.

>![](/images/network-doc/E15/Netstatports.png)

I proceed on setting the capture to port 53, which will capture any UDP or TCP packets coming through port 53.

>![](/images/network-doc/E15/capture1.png)

I set wireshark to capture, and do a dig+trace to student.labranet.jamk.fi.

>![](/images/network-doc/E15/digtracejamk.png)

## Analyzing the captured packets

>![](/images/network-doc/E15/capture1_resultlist.png)

Based on the filters we get a more managable result, which is expanded again below with some indicated areas. 

>![](/images/network-doc/E15/capture_resultmarked.png)

On the lefthand side of the list we see a list numbering each line more easily showing just the packets we have filtered. We can use this to go down chronologically. 
Within the info column we see the Transaction ID for the first query and query response. This is also indicated within the first query, as well as the actual query to student.labranet.jamk.fi.

From the initial query, the source ID is my VMs private address as source, and the destination-address (for the DNS query) is 199.7.83.42
Furthermore we see from the flags it is a standard query, and we see the response is listed in line no. 2. The arrows in the leftmost column also indicates which lines are in connection with eachother. 

A quick search on that destination-address gives us:

>![](/images/network-doc/E15/arinlookup.png){width=65%}

From the comment section it seems this might be a root server, run by the ICANN.

In the 2nd line, the response to 1 with transaction ID 0x22f9 is listed, as seen below.

>![](/images/network-doc/E15/captureresponse1.png){width=70%}

In this we see the response on the initial query with the authorative nameservers listed for top level domains (TLD) .fi, and the address to these.

**Continuing we see the next parts of the traceroute coming through as new DNS queries and requests.**

---

If we continue down the list on packets captured, we see that the next pair suddenly consist of IPv6 addresses. To try to identify these a bit closer, I need to go back up in the 2nd package, as well as checking my own IPv6.

>![](/images/network-doc/E15/FindingIpv6.png)

From here we see the source and destination IPs indicated with a blue square. From the terminal window we see my local machines IPv6, and in the list of authorative nameservers we do find further down the address in marked above in the blue square again, as c.fi type AAAA. 

Looking at the contents of the listed IPv6 Query and Response in No. 3 & 4, we get directed to a set of addresses with subdomains jazz, tango and humppa on the jamk domain. 

>![](/images/network-doc/E15/line4.png)

---

In lines 5 and 6 we see a request for jazz.jypoly.fi, which then gives an answer with the IP 195.148.128.19

>![](/images/network-doc/E15/answerjazz.png)

In the final line 5 and 6 of the traceroute we see that my machine is now requesting to a server humppa.jypoly.fi, which then responds with the address 195.148.26.130 for student.labranet.jamk.fi

>![](/images/network-doc/E15/line56.png)


- [x] Test and document dig towards a labranet address
	- [x] Capture packets and analyze

---

# Creating a local DNS list

*I change the network adapter back to internal on the Lubuntu1 VM, for the rest of the exercise. *

Before starting to add addresses and names as a local DNS, I ensure that Lubuntu1 has an IP and can reach the other parts of my network. 

>![](/images/network-doc/E15/Lubuntu1CheckingConnectivity.png)


## /etc/hosts

>![](/images/network-doc/E15/picohosts.png)

I proceed to add my lubuntu VMs - *including lubuntu1 which I am using which is uneccesary as I assume these will only be avaliable on the actual Lubuntu1 machine I am editing the file for*, the routers loopback address and the three switches to the file. 


>![](/images/network-doc/E15/hostsfile.png)


**SSH**:
>![](/images/network-doc/E15/testingssh.png)

**HTTP**:

>![](/images/network-doc/E15/routerahttp.png)

>![](/images/network-doc/E15/lubuntu1http.png)

**Ping**:

>![](/images/network-doc/E15/ping1.png)

**Traceroute**:

>![](/images/network-doc/E15/traceroute.png)

Directing and naming seems quite handy even with such a simple method as editing the hosts file. There are definitely benefits of knowing the subnets and addresses to each part of the network, but additionally adding "shortcuts" like these
to the network so I could more efficiently work and remember each part is a great tool. Even distributing the file to my small network could be a good idea! 

- [x] Create a local DNS list in /etc/hosts
	- [x] Testing, verifying and documenting