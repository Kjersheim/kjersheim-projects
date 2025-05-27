---
title: "Data Networks Episode 11: Measuring TCP and UDP"
date: 2023-03-18
draft: false
summary: "Exploring network performance and protocol behavior using iperf and Wireshark, including TCP 3-way handshake analysis and UDP-based DHCP discovery."
---

# Plan

- [ ] iperf
	- [ ] From home to the internet
	- [ ] Within my own network

- [ ] TCP 
	- [ ] Capture packets between Lubuntus and analyse 3-way-handshake

- [ ] UDP
	- [ ] Capture traffic between Lubuntu and Vyos to analyse the UDP part of DHCP 



# iperf

## From home to the internet

"iperf" is a command-line tool that is used in diagnosing network speed issues. It does this by measuring the maximum network traffick - back and forth - that a server can handle. It is particularily handy to use when you are having
network speed issues, as it can identify parts of the network where an problem might be located. 

After changing the network settings on one of my lubuntu machines I can test the iperf command as seen below.

>![](/images/network-doc/E11/iperf1.png)

This first setting is using iperf in client mode, connecting to the address iperf.he.net,
and as we can see from the output it shows three columns time interval, transfer amount and bandwidth. 

Changing the time to only 5 seconds, but still using the default port of 5001:

>![](/images/network-doc/E11/iperf4.png)

Changing the program to use UDP instead of TCP:

>![](/images/network-doc/E11/iperf2.png)

While TCP uses slow start, which is an algorithm which balances the speed of the network connection, to maximize bandwidth and minimize lost segments. By that it adapts to the slowest part of the network, and waits for the acks to come back. UDP leaves it to the 
application layer and does not wait for the acks. When specified, we can list what bandwidth we want to use with the -b / --bandwidth added to the command, and it runs even if too high - though with a considerable loss of datagrams.

```
From a discussion on Stack Exchange I find a decription to say that TCP segment is the protocol data unit which consists a TCP header and an application data piece (packet) which comes from the (upper) Application Layer. Transport layer data is generally named as a segment, 
and network layer data unit is named as a datagram, but when we use UDP as transport layer protocol we dont say UDP segment, but UDP datagram. This can be because we do not segmentate UDP data unit, but segmentation is made in the transport layer when we use TCP. 

Furthermore, I find that a TCP connection is a stream of bytes. The packetizing is done by the TCP/IP stack in the operating system. On the other hand, UDP is not a stream - its just a bunch of datagrams, that are not guaranteed to arrive in any order or arrive at all. Any protocol
implemented with UDP has to handle these details in their own application-specific way.

```

I chose a different public iperf server, actually located in Liev in Ukraine:

>![](/images/network-doc/E11/iperf5.png)

First TCP to the listed address on port 5201 and time set to 15 seconds. After that doing the same settings but using UDP.

>![](/images/network-doc/E11/iperf5_1.png)

Lastly I do a new test, where I notice that if I do not specify the 5201 port the connection is refused. In addition to that, UDP seems to be refused aswell. Maybe the given address only listens to a certain port, or the other ports somehow are busy. 
From what I understand these public servers can still only handle one request/iperf per port from users around the world at the time. 

I did initially try the bandwidth option on UDP but chose to wait with this till I can set up a server and client within my own network. 

- [ ] iperf
	- [x] From home to the internet
	- [ ] Within my own network

## Testing iperf within my own network

I undo the changes and keep Lubuntu1 away from the internet again, back in the comfort of the home network. I boot up Lubuntu2 as well and verify the connection between these two machines. 

>![](/images/network-doc/E11/ConnectionLub1and2.png)

For a reminder, the current physical and logical topology charts are as follows:

---
### Physical
>![](/images/network-doc/E11/E11NetworkCharts-PhysicalTopology.png){width=75%}

---
### Logical
>![](/images/network-doc/E11/E11NetworkCharts-LogicalTopology.png){width=75%}

---

>![](/images/network-doc/E11/iperfLub1to2.png)

First I set up the server with iperf -s (On Lubuntu1) and then do a default iperf from my Lubuntu2 to the server, seen above. After that I change some settings in the transmission, time to 30 seconds and bandwidth to 30M. It seems this is way too high, as
the result listed in the picture below (around 1.57 Mbits/sec) seems to be the highest it manages for that time. When I set the bandwidth down in other tests it does change according to the values I test. 

From the picture below you can see these tests, varying bandwidth results and rather short (5-30 seconds) tests, using the default port 5001 as a source (on the server side) and the client connecting from random ports. 

>![](/images/network-doc/E11/iperfLub1to2_2.png)

I set the interval to show information for each 2 seconds:

>![](/images/network-doc/E11/iperfLub1to2_3.png)

At the end of these tests I try to use UDP, but get an error I am not immediately understanding. It looks like some data is getting through, however I get a quite wide description later on that connection is refused. We did discuss the UDP not caring for ACK earlier, but still as seen below:

>![](/images/network-doc/E11/iperfLub1to2_4.png)

When then changing the settings on the server to listen to udp with the additional option -u also there, then sending from the client different transmissions varying from bandwidth of 15000k, 30000k and finally 400000K we see that in the last attempt the majority (78%) of the datagrams are lost. 

>![](/images/network-doc/E11/iperfLub1to2_5.png)

- [x] iperf
	- [x] From home to the internet
	- [x] Within my own network

## Investigating the connections with Wireshark

I add a wireshark VM between Lubuntu3 and Network Router B, as I intend to test traffic between Lubuntu2 and Lubuntu3. I continue to open a browser on Lubuntu2 from where I open the webserver on Lubuntu3. The following picture shows the 3-way-handshake from that connection.

>![](/images/network-doc/E11/3wayhandshaketcp.png)

Seen below is the first part of the **3-way-handshake**. From the IPv4 information we see the source is Lubuntu2 and destination Lubuntu3. 
When looking into the TCP header we can see which port the segment was sent from and received in. The latter shows port 80 which is used for http. 
In the bottom we see the bit describing "Syncronise" is enabled. 

>![](/images/network-doc/E11/part1_handshake.png)

In the next part, we see the ACK and SYN from Lubuntu3 to Lubuntu2. Lubuntu3 "acknowledges" Lubuntu2s request to "syncronise", and Lubuntu3 also sends a syncronise message back. At the end we see the bits for ACK and SYN have changed to 1. 
>![](/images/network-doc/E11/part2_handshake.png)

In the last part of this process we see the Lubuntu2 "acknowledges" Lubuntu3s request to "syncronise", and the connection is established. At the end we see Lubuntu3s acknowledgment bit is set to 1 in this segment sequence. 

>![](/images/network-doc/E11/part3_handshake.png)

- [x] TCP 
	- [x] Capture packets between Lubuntus and analyse 3-way-handshake


# Capturing DHCP traffic between Vyos and Lubuntu1

I proceed to connect the Wireshark Lubuntu between Lubuntu1 and Network Router A, on the network named LAN1, according to how I understood the exercise. While Lubuntu1 is offline I start capturing packets, before I start the Lubuntu1 again. 

![](/images/network-doc/E11/dhcp.gif)

As seen in the **gif** above, alot of descriptive traffic is captured. The DHCP request and acknowledgement containing the information to which addresses have been used previously, as well as subnet mask,  router information etc. 
However, from the UDP section we see the source port, destination port and length, as well as unverified checksum and a stream index. 

In the DHCP request we first see a broadcast from Lubuntu1 (mac-address 08:00:27:a3:bd:08) to the subnets broadcast address. Its source port is described as 68 and destination 67.
Shortly after as stream index 1, the router with address .1 responds to .5 and through the same ports vice verca and through the DHCP the address have been given. 

- [x] UDP
	- [x] Capture traffic between Lubuntu and Vyos to analyse the UDP part of DHCP 