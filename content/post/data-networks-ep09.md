---
title: "Data Networks Episode 9: Network segmentation and firewalls"
date: 2023-03-06
draft: false
summary: "Implementing policy-based segmentation using firewall zones in VyOS, including DHCP setup, OSPF integration, and rule-based traffic filtering."
---

## Plan


- [ ]  Prepare the topology
	- [ ] Update Physical schematic
	- [ ] Update Logical schematic
	- [ ] Add DHCP server on Router C to distribute addresses to the new public subnet 
	- [ ] Add new topology to the dynamic routing 

- [ ] Add and visualize 3 policy areas
- [ ] Configure firewall settings to Router C
- [ ] Test and verify firewall settings to Router C


# Preparing changes in topology

To prepare the added WAN and use of Router C as a firewall, I create an additional interface with this purpose.

>![](/images/network-doc/E09/NewInterfaceRouterC.png)

Looking through former exercises to find a suitable public IPv4 address space for testing, I find that the ones I researched earlier are mostly unresponsive. 
I chose to add **104.244.42.129** which is the address I get directed to when I ping twitter.com.

Before adding the address, I try to identify the subnet mask, in case thats required to add it. For testing purposes I will try to add without. Furthermore, trying to see
if there is a way this information is avaliable to the public I do a quick lookup on a slightly unsecured website:

>![](/images/network-doc/E09/Tryingtofindsubnetmask.png)

I add the address to the new interface on Vyos Router C. 

>![](/images/network-doc/E09/RouterCNewInterfaceEth2.png)

in the process I get info I do need to add the right format, aka. with a subnet-mask. However, as seen below.. any will do, so not sure what this does. 
I still chose finally to go with the supernet initially found. 
>![](/images/network-doc/E09/Needprefix.png)

>![](/images/network-doc/E09/AddingRouterCEth2Supernet.png)

---
### Note

I realize this is a huge supernet, but still just adding a few addresses in range from the dhcp server I set up on the router, for testing. Now, this makes me question some things. I am still barely grasping the functionality and processes of a network, so I take a step back. 
Thinking I can probably add a subnet from the information gathered above, and as of yet it is still only reachable on my own virtualized network. However, to avoid any issues and also chosing a smaller subnet I decide to pick one from my previous E03 Exercise:

---

|43.08.39.0/24|IP|bits|
|---|---|---|
|Address:|   43.08.39.0 |           00101011.00001000.00100111 .00000000|
|Netmask:|   255.255.255.0 = 24 |   11111111.11111111.11111111 .00000000|
|Wildcard:|  0.0.0.255   |          00000000.00000000.00000000 .11111111|
|Network:|   43.8.39.0/24  |        00101011.00001000.00100111 .00000000 (Class A)|
|Broadcast:| 43.8.39.255   |        00101011.00001000.00100111 .11111111|
|HostMin:|   43.8.39.1      |       00101011.00001000.00100111 .00000001|
|HostMax:|   43.8.39.254    |       00101011.00001000.00100111 .11111110|
|Hosts/Net:| 254|


**Changing interface 2 on Router C to have address: 43.8.39.1/24**

```
Adding 20 addresses for DHCP in the described subnet with the ethernet default-router as .1 and the addresses ranging from .5 to .25

set service dhcp-server shared-network-name publicwan subnet 43.8.39.0/24 default-router 43.8.39.1

set service dhcp-server shared-network-name publicwan subnet 43.8.39.0/24 range PCs start 43.8.39.5/24

set service dhcp-server shared-network-name publicwan subnet 43.8.39.0/24 range PCs stop 43.8.39.25/24
```

>![](/images/network-doc/E09/RouterCDhcpserver.png)

At last I add the subnet to the dynamic routing

>set protocols ospf area 0 network 43.8.39.0/24


- [x] Add DHCP server on Router C to distribute addresses to the new public subnet 
- [x] Add new topology to the dynamic routing 


Simultaneously, I have been working on the schematics for both the physical and logical topology. I make sure to update each one change and new devices/interface/address, as this is the best way to keep track of the changes and information I have and am gathering. 

I also clone a new workstation and connect that physically to the "public" internal network, to test from that side that the routing and dhcp services are working.

>![](/images/network-doc/E09/LubuntuWAN.png)

## Updated physical topology

![](/images/network-doc/E09/E09NetworkCharts-PhysicalTopology1.png)

## Updated logical topology

![](/images/network-doc/E09/E09NetworkCharts-LogicalTopology1.png)


- [x]  Prepare the topology
	- [x] Update Physical schematic
	- [x] Update Logical schematic

# Policy zones

I now am ready to declare the policy zones. As shown below, there will be three zones; WAN, Vyos and LAN.

![](/images/network-doc/E09/policyzones.png)

Initially I enable ping through the firewall that I will begin to set up shortly. I do this now, before declaring zones as suggested. 

>set firewall all-ping enable


I then declare the zones by the command
>set zone-policy zone <name> default-action <drop/accept>

Followed by adding interfaces to the zones, based on the firewall/router C. 
>set zone-policy zone <name> interface <interface>

![](/images/network-doc/E09/ZonePolicyAndInterfaces.png)

After this, committing changes and saving. Initially I tried saving after each command to be structured - however I was not allowed to add a zone without an interface. 

- [x] Add and visualize 3 policy areas

## Configuring the firewall

Before enabling the policy zones, I started a ping from Lubuntu WAN to another Lubuntumachine. The moment the zones were set, that ping was interrupted and all communication was unreachable. 
I thought initially that enable all-ping was about ping, but some internet research activites suggested otherwise. 

With this in mind, I also tried to figure out more thoroughly which and how to set the interfaces, and if the above was correct. The reason I questioned this, was due to the note in the materials mentioning to be precise when
adding interfaces, so VLAN would also spesicifally need to be added i.e. eth0.15 and such. However, I have no VLANS anywhere close to my router where the firewall is placed. By that note I also started thinking if I somehow needed
to add the zones to other devies. I can not find any information about this in the materials, so I chose to proceed to see where this lead me. 

When starting to add the rules, I had a thought of using ssh to configure the router a bit easier, but as there is no connection.. I will have to add them manually.

*I am not listing the commands here, but the guide used is summarized in the bottom of the documentation of this exercise, under section marked Notes. *

The rulesets added go from LAN to WAN, from LAN to vyos, from WAN to LAN, from WAN to vyos and lastly also from vyos to LAN and from vyos to WAN. The initial rulesets are more descriptive, while the returning is simply establishing stateful rules to accept the rules the other way too. 

![](/images/network-doc/E09/enablingfirewallrunes.gif)

*The rendering of this gif almost crashed my stationary office computer that is not ment to have 10 virtual machines open and in addition do actions it should not do to begin with, rendering.*

After adding the rulesets, I commit, save and reboot the router. At the same time I have a ping going between Lubuntu-WAN and Lubuntu1. When the router is rebooted, the ping is allowed through from Lubuntu1 and out to Lubuntu-WAN. However - it is not allowed from Lubuntu-WAN and into the network sections vyos or LAN. 

- [x] Configure firewall settings to Router C

## Verifying configuration

I open a browser on Lubuntu1, and enter the ip-address to Lubuntu WAN, showing the "Data Networks Course HTTP Server - It works!" webserver:

![](/images/network-doc/E09/Lubuntu1ToLWAN_HTTP_works.png)

Ping from Lubuntu2 to Lubuntu WAN:

![](/images/network-doc/E09/Lubuntu2ToWLAN_HTTP_and_ping_works.png)

Traceroute from Lubuntu3 to Lubuntu WAN, first as udp which gets delayed, and then specifying icmp that have been accepted and the route is finished:

![](/images/network-doc/E09/tracerouteLubuntu3_WAN_works.png)

I check statistics about ports from Lubuntu1 to Lubuntu WAN, with the Nmap scan technique "TCP SYN":

![](/images/network-doc/E09/NmapLubuntu1_WAN.png)

From the router I see several accepted packages, and some information about source, type and such:

![](/images/network-doc/E09/FirewallAcceptedLog.png)

There are also denied packages, like the one below which from what I can tell is the traceroute UDP request mentioned above from Lubuntu3 to Lubuntu WAN:

![](/images/network-doc/E09/FirewallLogDeniedL3Traceroute.png)

I wanted to go back a bit to figure out why the ping from Lubuntu WAN did not go through to Lubuntu1 after  the rulesets were added. 
I would not want someone to be able to access my devices like that, when it is an unknown actor. Security wise I get that it is as it should be. 

By going through the rulesets, initially I thought that allowing packages to be accepted back, having a stateful set of rules, would mean I could also ping from Lubuntu WAN
to Lubuntu1. I now understand that its the request echo that is accepted. If I would have wanteed to ping from WAN to LAN I would not only need to establish the state and relation, but also add
my own rule in wan to lan, and through vyos, declaring an action accept of protocol icmp.


- [x] Test and verify firewall settings to Router C
 
### Log
 - [Vyos Router C Firewall log, LAN_to_WAN](/images/network-doc/E09/firewalllogs.cfg)







            
# Notes

> perimeter defence around network segments

Filtering decisions within a firewall are typically installed in a 'firewall rule table'

Network segments typically means a sub/supernet. 'Policy' zones can contain several subnets within the zone, where all sub/supernets can share a policy.

Create an array - of a group/several subnets that can belong to the same cathegory of policies. "Categorization of network segments".
> Whitelist devices and groups that should have access, and deny anything else. 


**Stateful** Firewalls: 
The state table is a dynamic rule table, that reflects accepter traffics both directions. If something is accepted out, it is also accepted in.  Bidirectional.

**Stateless** need to both accept in AND out. Unidirectional.


>High availability (HA) allows you to place two firewalls in a group and synchronize their configuration. This prevents a single point of failure on your network. The two firewalls have a heartbeat connection, which ensures failover if one of the firewalls goes down
If firewalls are connected as such they might need a "heartbeat" cable to ensure syncronization of data. 



# Firewall rulesets
```
#Firewall: LAN->WAN
set firewall name LAN_to_WAN default-action drop
set firewall name LAN_to_WAN enable-default-log
set firewall name LAN_to_WAN rule 10 action accept
set firewall name LAN_to_WAN rule 10 destination port 80
set firewall name LAN_to_WAN rule 10 protocol tcp
set firewall name LAN_to_WAN rule 10 log enable
set firewall name LAN_to_WAN rule 11 action accept
set firewall name LAN_to_WAN rule 11 destination port 443
set firewall name LAN_to_WAN rule 11 protocol tcp
set firewall name LAN_to_WAN rule 11 log enable
set firewall name LAN_to_WAN rule 20 action accept
set firewall name LAN_to_WAN rule 20 protocol icmp
set firewall name LAN_to_WAN rule 20 log enable
set firewall name LAN_to_WAN rule 30 action accept
set firewall name LAN_to_WAN rule 30 destination port 22
set firewall name LAN_to_WAN rule 30 protocol tcp
set firewall name LAN_to_WAN rule 30 log enable
set firewall name LAN_to_WAN rule 40 action accept
set firewall name LAN_to_WAN rule 40 destination port 53
set firewall name LAN_to_WAN rule 40 protocol udp
set firewall name LAN_to_WAN rule 40 log enable

#Firewall: WAN->LAN
set firewall name WAN_to_LAN default-action drop
set firewall name WAN_to_LAN rule 10 action accept
set firewall name WAN_to_LAN rule 10 state established enable
set firewall name WAN_to_LAN rule 10 state related enable

#Firewall: LAN->vyos
set firewall name LAN_to_vyos default-action drop
set firewall name LAN_to_vyos rule 10 action accept
set firewall name LAN_to_vyos rule 10 destination port 22
set firewall name LAN_to_vyos rule 10 protocol tcp
set firewall name LAN_to_vyos rule 20 action accept
set firewall name LAN_to_vyos rule 20 protocol ospf

#Firewall: vyos->LAN
set firewall name vyos_to_LAN default-action drop
set firewall name vyos_to_LAN rule 10 action accept
set firewall name vyos_to_LAN rule 10 state established enable
set firewall name vyos_to_LAN rule 10 state related enable
set firewall name vyos_to_LAN rule 20 action accept
set firewall name vyos_to_LAN rule 20 protocol ospf

#Firewall: vyos->WAN
set firewall name vyos_to_WAN default-action drop
set firewall name vyos_to_WAN rule 10 action accept
set firewall name vyos_to_WAN rule 10 state established enable
set firewall name vyos_to_WAN rule 10 state related enable

#Firewall: WAN->vyos
set firewall name WAN_to_vyos default-action drop
set firewall name WAN_to_vyos rule 10 action accept
set firewall name WAN_to_vyos rule 10 state established enable
set firewall name WAN_to_vyos rule 10 state related enable
set firewall name WAN_to_vyos rule 20 action accept
set firewall name WAN_to_vyos rule 20 protocol icmp

#Finally binding the firewall rules to traffic between the zones
set zone-policy zone LAN from WAN firewall name WAN_to_LAN
set zone-policy zone WAN from LAN firewall name LAN_to_WAN
set zone-policy zone vyos from LAN firewall name LAN_to_vyos
set zone-policy zone LAN from vyos firewall name vyos_to_LAN
set zone-policy zone WAN from vyos firewall name vyos_to_WAN
set zone-policy zone vyos from WAN firewall name WAN_to_vyos

```