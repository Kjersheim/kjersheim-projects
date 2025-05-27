---
title: "Data Networks Episode 2: First switches"
date: 2023-01-28
draft: false
summary: "Setting up and configuring two virtual switches with VLANs and monitoring traffic using Wireshark."
---

# E02 First switches

# Introduction 

## Setting up:

Initially updating my lubuntu-box, and then importing my first switch. Making clones of both, and making sure to name and create new MAC for all boxes/devices. 

The lubuntu boxes, named Lubuntu 1 and Lubuntu 2, have 1 network adapter. These are both directed to an internal network, pointing to LAN 1 and LAN 3, respectively.

I start configuring the networks of the two switches, named Network_Switch_A and Network_Switch_B. 

```
Both of them have Adapter 1 set to host-only, for management. 
Second, the Adapter 2 and 3 is pointed to for Switch A LAN 1 and 2 (Back to Lubuntu 1 and to what will be the line to Switch B) 
and finally in Switch B - Adapter 2 and 3 respectfully LAN 2 and 3(finishing the connection to Switch A, and to Lubuntu 2)
```

The first network diagram at this point:  
![](/images/network-doc/E02/E02NetworkChart_01.jpg)

Setting the IP and gateway manually — checking that it's active — and then trying the first ping between Lubuntu 1 and 2:  
![](/images/network-doc/E02/CheckingIpAddresses.png)  
![](/images/network-doc/E02/CheckingConnectivity.png)

---

## Configuring the switches:

On both switches I follow the instructions to create the workstation VLAN, and re-adding the ports so they are assigned to the newly created VLAN. Finally, I tag the port for the chosen adapter which is going to the switch, and untag the port used for the adapter to the workstations. 

It took me a few tries trying to figure out which adapter was used for which port… surprisingly… until I figured out it was quite consecutively added.  
From the website [networkdirection.net](https://networkdirection.net) and an article regarding tagged and untagged ports, I gathered more insight, also supported by our course FAQ.  

```
Trying to summarize my findings for future sourcing:

A VLAN is a group of devices that share a broadcast area. A VLAN can separate different parts of say a building or company, based on whatever is logical, physical or requirements based on department and who to talk to who. 

From my understanding so far VLANs are isolated, so devices within a VLAN can only communicate with other devices within the same VLAN. But..

Untagged — or access VLANs — are connected to hosts (often a server) that pass VLAN information to and from each network.  
Tagged — or Trunk VLANs — enable switch access ports to handle more than 1 VLAN and separate traffic accordingly.  
Instead of data going from one host to another, the frames with a VLAN tag can be distributed from one host to many other hosts, that are connected to a port — depending on their configuration.  
The tags decide which packets should be sent to specific VLANs on the other side.

* Stronger security – identifier tags  
* Less congestion – preconfiguring traffic directions  
* Visibility – Easier to understand and see issues
```

Adding the configurations to Switch A:  
![](/images/network-doc/E02/NetworkSwitchAVlanConfig.png)

Doing the same for Switch B, but making sure to tag and untag the correct ports, making the current version of the network diagram:  
![](/images/network-doc/E02/E02NetworkChart_02.jpg)

---

# Wireshark box

## Capturing network data on LAN 1

I clone another lubuntu box and name it **Lubuntu Wireshark**. I set its adapter 1 to Internal Network and LAN 1 — as I want to initially listen to that connection.

Booting up the box and starting Wireshark, then doing a ping from Lubuntu 1 (which is on LAN1) to Lubuntu 2, and also pinging Lubuntu 1 from Lubuntu 2. From the Wireshark scan I see both requests and replies.

I stop the recording and save it locally on the Lubuntu Wireshark box:  
![](/images/network-doc/E02/FirstCaptureSaveLocation.png)

Continuing by opening the capture and choosing a frame to have a closer look:  
![](/images/network-doc/E02/ChosenFrame.png)

From the information visible in the main screen of Wireshark before selecting the frame, I see it's a ping request from Lubuntu 1 to Lubuntu 2.

Selecting the frame gives further detailed information:  
![](/images/network-doc/E02/FrameInfo1.png)

The EthType is also marked with a red square, IPv4 (0x0800) which gives receiving and transmitting party the information of what the frame contains.

The source IP is marked with a red square connected with a red line. The source MAC address is also marked with a red square, and it can be seen as the same as within the network settings of Lubuntu 1:  
![](/images/network-doc/E02/Lubuntu1Mac.png)

Finally, trying to identify the frame's payload — which I understand as the data that is being carried by the frame. From what I can find, it contains the OSI model's Layer 3 header (IP), Layer 4 header (TCP), and the application data:  
![](/images/network-doc/E02/FrameInfo2.png)

---

## Changes on Network_Switch_A

Configuring Network_Switch_A with an IP and allowing SSH connections:  
![](/images/network-doc/E02/SwitchSetIPAllowSSH.png)

From Lubuntu 2 I SSH into Network_Switch_A, and run `show configuration`, results shown below:  
![](/images/network-doc/E02/SSHFromLubuntu2ToSwitchA.png)

```plaintext
<Switch A configuration output shown here...>
(enable ssh2, VLAN setup, etc.)
```

---

## Changes on Network_Switch_B

It’s not entirely clear which devices were referred to as “both devices” in the task, but I assume it means to repeat the same setup on the other switch.

I do the exact same configuration on Network_Switch_B, choosing a different IP, and log into the switch from Lubuntu 1:  
![](/images/network-doc/E02/LastConfigSwitchB.png)

```plaintext
<Switch B configuration output shown here...>
```

Final version of the network diagram, after giving the switches specific IPs:  
![](/images/network-doc/E02/E02NetworkChart_03.jpg)
