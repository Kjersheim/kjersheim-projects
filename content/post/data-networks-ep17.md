---
title: "Data Networks Episode 17: Making the Network Public"
date: 2023-05-02
draft: false
summary: "In this final episode, the lab network is connected to the public internet by bridging Router 3 to the home network. Tasks include identifying local and public IP ranges, updating topology charts, configuring routes and DNS, and verifying connectivity through real-world web and traceroute tests. The episode concludes the full configuration and testing cycle of the lab network."
---

# Plan

- [ ] Investigate and document home networks IPv4 address range
- [ ] Add info and update topology charts

- [ ] Router3 
	- [ ] Configurate interface to bridge to the internet
	- [ ] Verify DNS configurations
	- [ ] Add a default route from the router to the internet

- [ ] Update DHCP servers

- [ ] Verification and connectivity

- [ ] Update configuration files with E17 changes

# Current home network

## Private

Bridging the Lubuntu Wireshark machine to the hostmachine for a brief check of the private network.

The gateway aka the private address to the router in my home network is as follows:

>![](/images/network-doc/E17/PrivateGateway.png)

Checking if other devices are connected:

>![](/images/network-doc/E17/hosts_alive_private.png)

Asking for a bit more detail, I search for any devices with web-related ports open. On a sidenote, could be an interesting way to for example check for security issues within my network when ports are not supposed to be open. 

>![](/images/network-doc/E17/Hosts_scan.png)

Lastly, adding the interfaces on the router:

>![](/images/network-doc/E17/routeriface.png)

## Public

The first step I take is to go to www.ripe.net and check which address my router is using to be in contact with the internet. The following shows the range, although I have censored out the last octet for privacy reasons. 

>![](/images/network-doc/E17/routerip.png)

A quick query for the address gives the following information. Noted in the category origin, we find an autonomous system 
registred on the ISP DNA. 

>![](/images/network-doc/E17/DNA_subnet.png)

>![](/images/network-doc/E17/DNA_route.png)

## AS16086 - DNA

From the IP search initially I find information about the DNA AS I am connected to. Some information tells me what kind of services the
AS provides, for example a network described as "eyeball" which I find is a slang to categorise access networks whose primary users use the network to "look at things"..

In addition to that there is a range of prefixes, other supernets, from which I understand is within the control of the AS in question, as well as a range of neighbours and other AS.

>![](/images/network-doc/E17/AS16086_1.png)

>![](/images/network-doc/E17/Peering_db_AS16086.png)

>![](/images/network-doc/E17/AS16086_IX.png)

## AS16086 IPv4 Route Propagation

>![](/images/network-doc/E17/AS16086_RoutePropagation1.png)\
*Source: https://bgp.he.net/AS16086*

Looking into the map above, we find the AS174 as a system within Arin in North-America by Cogent Communications, AS2119 a Norwegian ISP Telenor, AS3356 by Lumen and located mostly in Northern America but also other regions of the world. 

- [x] Investigate and document home networks IPv4 address range

# Updated physical and logical topology schematics

## Physical

>![](/images/network-doc/E17/E17NetworkCharts-PhysicalTopology.png)

## Logical 

>![](/images/network-doc/E17/E17NetworkCharts-LogicalTopology1.png)

- [x] Add info and update topology charts


# Configuration changes in Vyos Router 3

Initially I power off the Vyos Router 3 to change its Adapter 3 to bridged interface. Once that is done, I remove the old public WAN address starting with 43, as well as the IPv6 address I added in E16. The bridged interface is registred to be
set to the previously chosen interface that have the NAT configuration as outbound interface. 

>set protocols static route 0.0.0.0/0 next-hop 192.168.1.1
>set protocols ospf default-information originate metric-type 2

- [x] Router3 
	- [x] Configurate interface to bridge to the internet
	- [x] Verify DNS configurations
	- [x] Add a default route from the router to the internet

## Setting DNS 
Finally I set the DNS-server to 1.1.1.1 on all three routers within their dhcp settings, for each subnets thats connected to the given router. 

>set service dhcp-server shared-network-name <name> subnet <subnet> dns-server 1.1.1.1
>as one example on Router1 names would be Lubuntu1 and Lubuntu2, and the subnets would be the subnets the Lubuntu1 and Lubuntu2 are within. 

- [x] Update DHCP servers


# Verification and connectivity

From Lubuntu1 I perform a series of pings and traceroutes to see the effect of the changes.

Checking its own IP its still within the virtual network created over the last few weeks.

>![](/images/network-doc/E17/lub1ip.png)

Pinging home router:

>![](/images/network-doc/E17/lub1router.png)

Using an icmp echo as the method for traceroute -I to the router:

>![](/images/network-doc/E17/TracerouteI.png)

Using the browser on Lubuntu1 to access NRK, the norwegian equivalent to YLE:

>![](/images/network-doc/E17/Lub1NRK.png)

Running a ping from Switch C to the home router at 192.168.1.1, and another ping towards the IP behind www.yle.fi:

>![](/images/network-doc/E17/SwitchCping.png)

Performing a traceroute to the same YLE ip-address:
>![](/images/network-doc/E17/Switchctraceroute.png)

- [x] Verification and connectivity


## Updated configuration-files after completing E17

[Network_Switch_A](/images/network-doc/E17/Config_files/E17-SwitchAu.cfg)\
[Network_Switch_B](/images/network-doc/E17/Config_files/E17-SwitchBu.cfg)\
[Network_Switch_C](/images/network-doc/E17/Config_files/E17-SwitchCu.cfg)

[Vyos_Router_A](/images/network-doc/E17/Config_files/E17-RouterAu.cfg) \
[Vyos_Router_B](/images/network-doc/E17/Config_files/E17-RouterBu.cfg) \
[Vyos_Router_C](/images/network-doc/E17/Config_files/E17-RouterCu.cfg) 

- [x] Update configuration files with E17 changes


End note: Thank you for an outstanding course. Once the remaining lecture videos are done in english it wil be even more fulfilling. Have a great summer.