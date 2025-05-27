---
title: "Data Networks Episode 8: Loop detection, part 2. IPv4, routing"
date: 2023-02-28
draft: false
summary: "Enabling dynamic routing with OSPF in a virtual lab environment, including topology updates, router configuration, and security considerations."
---

## Plan

- [ ] Update topology
	- [ ] Physical schematic
	- [ ] Logical schematic

- [ ] Update & add virtual machines
	- [ ] Configuration of new router
	- [ ] Loopback interfaces

- [ ] Configure OSPF on routers
	- [ ] Dynamic routing
	- [ ] Checking for routing table
	
- [ ] Connectivity tests

- [ ] Configure cybersecurity to OSPF


# Updating topology

## Physical topology
![](/images/network-doc/E08/E08NetworkCharts-PhysicalTopologyDraft2.png)

## Logical topology
![](/images/network-doc/E08/E08NetworkCharts-LogicalTopologyDraft2.png)



- [x] Update topology
	- [x] Physical schematic
	- [x] Logical schematic



# Update & add virtual machines
Subnet No. 1

10.08.83.244/30 (0100) Between Vyos router B and C


Subnet No. 2

10.08.83.248/30 (1000) Between Vyos router A and B



Renaming Vyos Router C, initial configuration:

```	
'configure
	set system host-name [name]
	commit
	save
	exit'

Requires a full reboot of the Vyos virtual machine to take affect.
'poweroff 
y'

Checking interfaces on Vyos Router C:
'show configuration'

comparing interfaces to the adapters on the virtual machine. Updating logical and physical topology. 


Adding ip-addresses to interfaces:

Since I dont have any VLANS within the two new subnets, I set the ip address directly on the interfaces and not adding the vif option. 

' set interfaces ethernet eth3 address 10.8.83.246/30
  set interfaces ethernet eth0 address 10.8.83.250/30 
  commit
  save '

```	

![](/images/network-doc/E08/RouterCAddresses.png)

Eventually I check the addresses and add them to each interface eth0 and eth3. 

I chose to delete the previous established static routes that were cloned from the original router. Not sure if this was neccesary if we are adding dynamic routing, but I think they would nomatter be wrong. 

![](/images/network-doc/E08/DeletingRoutes.png)


Finally I check the configuration of the new interfaces on Vyos Router A and B, and add them to the topologies. 

![](/images/network-doc/E08/NewCablesRouterAB.png)

From this I start a test pinging between routers. 
- Ping between Router A and B is positive. 
- Ping between Router B and C is positive. 
- Ping between Router A and C is positive. 

![](/images/network-doc/E08/InitialRouterPingTests.png)

Ping outside of each subnet however is not yet possible, as no gateways or routing has been set. 

- [ ] Update & add virtual machines
	- [x] Configuration of new router
	- [ ] Loopback interfaces

	

# Addressing loopback interfaces

Initially I am trying to understand where the loopback lo would belong within the OSI model. As seen below, the loopback interface is linked to a certain ip-address, so in that sense I assume its a part of 
the network layer 3. However doing some searches on the matter, I find answers also describing that there isnt a clean separation of the two TCP and IP-stacks.  Having these working on multiple elements of 
the network layer, the transportation layer and also the session layer. Furthermore, loopback processing might also be done in a separate device driver that for the OS looks like any other device, and possibly even be looked
at as a part of the physical layer? Several answers describe it in the way that the OSI model might be used for describing the protocols themselves, but when it comes to ths structure of the network code its harder to place it in a singular location of the model. 

## Describing the subnets to use
```
"parent" subnet:
10.08.83.252/30
```

Approaching this made me need to step back to have a look on previous made subnets and the lessons I have learned so far. We could say that the /30 or /32 does not refer to the size of the 
subnet, but rather the range within. Thinking like this makes more sense, but still does not explain the regular information we have about subnets so far. 
Quite clearly there is no room for a subnet- or broadcastaddress within. /32 would have a subnet-mask of 255.255.255.255, and would only contain one single host. 
From some research I find that these are used for specific purposes, as loopback interfaces or a "host route". 

On my Vyos Router A I can see that 127.0.0.1 is refered to as link/loopback:

![](/images/network-doc/E08/routeripaddress.png)

Previously I am familiar with this address described as "localhost", or the machine itself. Doing a quick search also shows the description that
the loopback address refers to the fact that no data packet addressed to 127.0.0.1 should leave the computer/host, 
and sending it the packet gets "looped back" to on itself, so the sender becomes the recipient. 

For example, previously we used an extension on Vistual Studio Code to host a webserver locally, on 127.0.0.1 :5500.


RFC1122 refers to the address 127.0.0.1 as a internal host loopback address, and no other addresses of this form must appear outside the host. 
Routers that pick up traffic from the address are supposed to drop the packets immidiately. Whilst trying to focus on the framework of the assignment,
it is curious to consider the use/purpose/plan for the rest of the addresses within 127.0.0.1 to 127.255.255.255, where the whole block is reserved.

Continuing on the exercise I believe the subnet mentioned above can be divided. Since I need three of the total 4 addresses avaliable, Ill describe those. 


- 10.8.83.253/32 - Vyos Router A
- 10.8.83.254/32 - Vyos Router B
- 10.8.83.255/32 - Vyos Router C


![](/images/network-doc/E08/RouterLoopbackAddresses.png)

- [x] Update & add virtual machines
	- [x] Configuration of new router
	- [x] Loopback interfaces


# Configuring OSPF on the routers

Initially I delete the old static routes from router A and B. Previously I already deleted them on the cloned router C, when changing its configurations. 

I continue setting the router-id manually to the loopback interfaces I previously added, on all 3 devices and using the addresses referred to above:

![](/images/network-doc/E08/settingrouterid.png)


At this point I needed to take a step back, trying to figure out which exact subnets to add as ospf to within each router. 
Initially I only added the two subnets each router was connected to - between themselves. Completely forgetting the other workstation-groups, switches and loopback interfaces. 
I then realized that is not going to be enough, as they wont magically transfer the ospf further out in the area.

**The commands used at this point was set protocols ospf area 0 network -subnet-address-**

For each router, I add the loopback interface I previously set to them. In addition to that I kept the two subnets I already added to each router. 
Lastly I added the subnets that router A and Router B was connected to, and that I had not yet added.

So Router A would be a main point, connected to the following:

![](/images/network-doc/E08/VyosRouterAospf1.png)

at this point I was wondering about the "greetings" between adjacent devices. As depicted in the picture below, both router A and router B will be
 sending information about the subnet .240/30. Is this somehow going to affect any processes in the routing? 

![](/images/network-doc/E08/Question1.png)

In other words, if only router A is sharing information about the subnet between router A and B, it will still be known in the network topology, or vice verca if only router B is sharing its information.
Furhtermore, I was thinking what if a router would be disconnected or having issues, would it be beneficial of that subnet then to have another "broadcaster"? Then again, if router A goes down, the subnet between router A and B have little to no function. 
The end question I guess would be if it is an issue that two routers are sharing the information, if it would get duplicated somehow. Yet to find an answer to this, as I decide to add all subnets connected to each router. 

![](/images/network-doc/E08/RouterBOspf.png)
![](/images/network-doc/E08/RouterCOspf.png)


## OSPF status & information checks

Logging into the routers via ssh to easier copy the information, as that would be searchable and easier to maintain than screenshots. 

:zap:
Router A


```
**********************
show ip ospf neighbor*
**********************
vyos@vyos-router-a:~$ show ip ospf neighbor 

Neighbor ID     Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
10.8.83.255       1 Full/DR           37.395s 10.8.83.250     eth2:10.8.83.249         0     0     0
10.8.83.254       1 Full/DR           30.980s 10.8.83.242     eth1:10.8.83.241         0     0     0


*************************************
show ip route ospf and show ip route*
*************************************
vyos@vyos-router-a:~$ show ip route ospf 
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O   10.8.83.0/28 [110/10] is directly connected, eth0.25, 01:14:43
O   10.8.83.240/30 [110/1] is directly connected, eth1, 01:14:09
O>* 10.8.83.244/30 [110/2] via 10.8.83.242, eth1, 01:14:07
O   10.8.83.248/30 [110/1] is directly connected, eth2, 01:14:08
O   10.8.83.253/32 [110/0] is directly connected, lo, 01:14:55
O>* 10.8.83.254/32 [110/1] via 10.8.83.242, eth1, 01:14:07
O>* 10.8.83.255/32 [110/1] via 10.8.83.250, eth2, 01:14:04
O   192.168.39.0/26 [110/10] is directly connected, eth0.5, 01:14:42
O   192.168.39.64/26 [110/10] is directly connected, eth0.15, 01:14:42
O>* 192.168.39.128/26 [110/11] via 10.8.83.242, eth1, 01:14:07

vyos@vyos-router-a:~$ show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O   10.8.83.0/28 [110/10] is directly connected, eth0.25, 01:15:08
C>* 10.8.83.0/28 is directly connected, eth0.25, 01:15:23
O   10.8.83.240/30 [110/1] is directly connected, eth1, 01:14:34
C>* 10.8.83.240/30 is directly connected, eth1, 01:15:15
O>* 10.8.83.244/30 [110/2] via 10.8.83.242, eth1, 01:14:32
O   10.8.83.248/30 [110/1] is directly connected, eth2, 01:14:33
C>* 10.8.83.248/30 is directly connected, eth2, 01:15:21
O   10.8.83.253/32 [110/0] is directly connected, lo, 01:15:20
C>* 10.8.83.253/32 is directly connected, lo, 01:15:25
O>* 10.8.83.254/32 [110/1] via 10.8.83.242, eth1, 01:14:32
O>* 10.8.83.255/32 [110/1] via 10.8.83.250, eth2, 01:14:29
O   192.168.39.0/26 [110/10] is directly connected, eth0.5, 01:15:07
C>* 192.168.39.0/26 is directly connected, eth0.5, 01:15:22
O   192.168.39.64/26 [110/10] is directly connected, eth0.15, 01:15:07
C>* 192.168.39.64/26 is directly connected, eth0.15, 01:15:22
O>* 192.168.39.128/26 [110/11] via 10.8.83.242, eth1, 01:14:32



**********************
show ip ospf database*
**********************
vyos@vyos-router-a:~$ show ip ospf database 

       OSPF Router with ID (10.8.83.253)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
10.8.83.253     10.8.83.253     1080 0x8000001b 0x0ee9 6
10.8.83.254     10.8.83.254     1100 0x80000011 0xbfe1 4
10.8.83.255     10.8.83.255     1048 0x8000000e 0xf57d 3

                Net Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum
10.8.83.242     10.8.83.254     1070 0x80000003 0xf6df
10.8.83.246     10.8.83.255     1088 0x80000003 0xe0ee
10.8.83.250     10.8.83.255     1069 0x80000003 0xae1e


```


:zap:
Router B


```
**********************
show ip ospf neighbor*
**********************
vyos@vyos-router-b:~$ show ip ospf neighbor 

Neighbor ID     Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
10.8.83.253       1 Full/Backup       39.091s 10.8.83.241     eth1:10.8.83.242         0     0     0
10.8.83.255       1 Full/DR           35.042s 10.8.83.246     eth0:10.8.83.245         0     0     0



*************************************
show ip route ospf and show ip route*
*************************************
vyos@vyos-router-b:~$ show ip route ospf 
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O>* 10.8.83.0/28 [110/11] via 10.8.83.241, eth1, 01:25:58
O   10.8.83.240/30 [110/1] is directly connected, eth1, 01:26:44
O   10.8.83.244/30 [110/1] is directly connected, eth0, 01:25:54
O>* 10.8.83.248/30 [110/2] via 10.8.83.241, eth1, 01:25:45
  *                        via 10.8.83.246, eth0, 01:25:45
O>* 10.8.83.253/32 [110/1] via 10.8.83.241, eth1, 01:25:58
O   10.8.83.254/32 [110/0] is directly connected, lo, 01:26:44
O>* 10.8.83.255/32 [110/1] via 10.8.83.246, eth0, 01:25:45
O>* 192.168.39.0/26 [110/11] via 10.8.83.241, eth1, 01:25:58
O>* 192.168.39.64/26 [110/11] via 10.8.83.241, eth1, 01:25:58
O   192.168.39.128/26 [110/10] is directly connected, eth2, 01:26:44


vyos@vyos-router-b:~$ show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O>* 10.8.83.0/28 [110/11] via 10.8.83.241, eth1, 01:26:19
O   10.8.83.240/30 [110/1] is directly connected, eth1, 01:27:05
C>* 10.8.83.240/30 is directly connected, eth1, 01:27:07
O   10.8.83.244/30 [110/1] is directly connected, eth0, 01:26:15
C>* 10.8.83.244/30 is directly connected, eth0, 01:26:59
O>* 10.8.83.248/30 [110/2] via 10.8.83.241, eth1, 01:26:06
  *                        via 10.8.83.246, eth0, 01:26:06
O>* 10.8.83.253/32 [110/1] via 10.8.83.241, eth1, 01:26:19
O   10.8.83.254/32 [110/0] is directly connected, lo, 01:27:05
C>* 10.8.83.254/32 is directly connected, lo, 01:27:11
O>* 10.8.83.255/32 [110/1] via 10.8.83.246, eth0, 01:26:06
O>* 192.168.39.0/26 [110/11] via 10.8.83.241, eth1, 01:26:19
O>* 192.168.39.64/26 [110/11] via 10.8.83.241, eth1, 01:26:19
O   192.168.39.128/26 [110/10] is directly connected, eth2, 01:27:05
C>* 192.168.39.128/26 is directly connected, eth2, 01:27:10


**********************
show ip ospf database*
**********************
vyos@vyos-router-b:~$ show ip ospf database 

       OSPF Router with ID (10.8.83.254)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
10.8.83.253     10.8.83.253       28 0x8000001c 0x0cea 6
10.8.83.254     10.8.83.254       86 0x80000012 0xbde2 4
10.8.83.255     10.8.83.255       16 0x8000000f 0xf37e 3

                Net Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum
10.8.83.242     10.8.83.254       46 0x80000004 0xf4e0
10.8.83.246     10.8.83.255       56 0x80000004 0xdeef
10.8.83.250     10.8.83.255        6 0x80000004 0xac1f


```


:zap:
Router C


```
**********************
show ip ospf neighbor*
**********************
vyos@vyos-router-c:~$ show ip ospf neighbor 

Neighbor ID     Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
10.8.83.254       1 Full/Backup       34.546s 10.8.83.245     eth3:10.8.83.246         0     0     0
10.8.83.253       1 Full/Backup       36.752s 10.8.83.249     eth0:10.8.83.250         0     0     0


*************************************
show ip route ospf and show ip route*
*************************************
vyos@vyos-router-c:~$ show ip route ospf 
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O>* 10.8.83.0/28 [110/11] via 10.8.83.249, eth0, 01:29:21
O>* 10.8.83.240/30 [110/2] via 10.8.83.249, eth0, 01:29:21
O   10.8.83.244/30 [110/3] via 10.8.83.249, eth0, 01:29:21
O   10.8.83.248/30 [110/1] is directly connected, eth0, 01:30:10
O>* 10.8.83.253/32 [110/1] via 10.8.83.249, eth0, 01:29:21
O>* 10.8.83.254/32 [110/2] via 10.8.83.249, eth0, 01:29:21
O   10.8.83.255/32 [110/0] is directly connected, lo, 01:30:15
O>* 192.168.39.0/26 [110/11] via 10.8.83.249, eth0, 01:29:21
O>* 192.168.39.64/26 [110/11] via 10.8.83.249, eth0, 01:29:21
O>* 192.168.39.128/26 [110/12] via 10.8.83.249, eth0, 01:29:21

yos@vyos-router-c:~$ show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O>* 10.8.83.0/28 [110/11] via 10.8.83.249, eth0, 01:29:37
O>* 10.8.83.240/30 [110/2] via 10.8.83.249, eth0, 01:29:37
O   10.8.83.244/30 [110/3] via 10.8.83.249, eth0, 01:29:37
C>* 10.8.83.244/30 is directly connected, eth3, 01:30:34
O   10.8.83.248/30 [110/1] is directly connected, eth0, 01:30:26
C>* 10.8.83.248/30 is directly connected, eth0, 01:30:26
O>* 10.8.83.253/32 [110/1] via 10.8.83.249, eth0, 01:29:37
O>* 10.8.83.254/32 [110/2] via 10.8.83.249, eth0, 01:29:37
O   10.8.83.255/32 [110/0] is directly connected, lo, 01:30:31
C>* 10.8.83.255/32 is directly connected, lo, 01:30:36
O>* 192.168.39.0/26 [110/11] via 10.8.83.249, eth0, 01:29:37
O>* 192.168.39.64/26 [110/11] via 10.8.83.249, eth0, 01:29:37
O>* 192.168.39.128/26 [110/12] via 10.8.83.249, eth0, 01:29:37


**********************
show ip ospf database*
**********************
vyos@vyos-router-c:~$ show ip ospf database 

       OSPF Router with ID (10.8.83.255)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
10.8.83.253     10.8.83.253      231 0x8000001c 0x0cea 6
10.8.83.254     10.8.83.254      290 0x80000012 0xbde2 4
10.8.83.255     10.8.83.255      217 0x8000000f 0xf37e 3

                Net Link States (Area 0.0.0.0)

Link ID         ADV Router      Age  Seq#       CkSum
10.8.83.242     10.8.83.254      250 0x80000004 0xf4e0
10.8.83.246     10.8.83.255      257 0x80000004 0xdeef
10.8.83.250     10.8.83.255      207 0x80000004 0xac1f


```

- [x] Configure OSPF on routers
	- [x] Dynamic routing
	- [x] Checking for routing table
	
# Verifying connectivity

From the show ip commands printed above, and preliminary ping testing, devices on the network are identified and can reach eachother. To ensure all devices are online and also checking routes, I perform some tests with ping and traceroute.

A request from Lubuntu1 to Lubuntu3 is sent towards router A, which is the gateway in Lubuntu1s subnet. From there it continues to router B, which is the gateway for Lubuntu3s subnet. Last stretch is straight to Lubuntu3.

![](/images/network-doc/E08/RouterL1L3.png)

Next I check the route between Lubuntu1 and Router C. First it goes directly, to A then C. If I then chose the address to router C, but in the subnet between Router C and B - it goes via Router B. 

![](/images/network-doc/E08/tracerouteL1RouterC.png)


### Reference-bandwidth

At this point I also change the reference bandwidth to cost value in all 3 routers. 

set protocols ospf auto-cost reference-bandwidth 1000

Setting it up from 100 to 1000 Mbit/sec. Committing, saving and restarting the routers. 
Despite not expecting to see any changes, as I cant see any way to imitate a cost difference on the virtual network as of this point,
 I still want to set it up due to having gigabit ethernet lines on my "physical plan" in E06. 
 
I do another test just to ensure there are no changes, and the routes described in the above picture depicting the route between Lubuntu1 and RouterC remains the same. 

Furthermore, I did consider incrasing it even more, this due to:

> Ethernet description   -- Reference bandwidth / default bandwidth = cost

> Gigabit ethernet 1 Gbps -- 1 000 000 000 / 1 000 000 000 = 1 

> 10 Gigabit ethernet 10 Gbps -- 1 000 000 000 / 10 000 000 000 = 1

> Fast ethernet 100Mbps -- 1 000 000 000 / 100 000 000 = 10


As seen, even with 1000 Mbps reference, 1 and 10 Gbps would be given the same cost. 

Increasing the reference-bandwidthto 10 000 then could solve this, but for now I leave it at 1000 as my net is not have that fast infrastructure yet, 
and that the network devices and cabling would be well within and below 1 Gbps in most areas, except between the main devices which are planned with the 1Gbps lines. 


### Connectivity and analysis continues

I remember from the materials to be wary of the SPF execution count. By doing a check on all three routers, I find the SPF algoritmn has run 19, 14 and 11 times on A, B and C. I cant really tell how this is counted, but I assume it
has to do with changes in the network - and perhabs also at a certain time to make sure it gets the new information. I am yet to fully delve into the different parts of the packet exchange between the routers, but 

> show ip ospf database 

Previously I ran the abovementioned ospf data command, which shows that all 3 routers are visible in this main(and at the moment only) area, the backbone. 
It also shows how many links each router has, still within the area. Router A, as we know but also as seen with the command, with 6 links as situated quite in the middle and being connected to more subnets than the others. 

![](/images/network-doc/E08/databaseroutera.png)

> show ip ospf neighbour

From the information about how the routers sees its neighbours, we can tell that:

- For Router A perspective, RouterB and RouterC is the designated router. 
- For Router B, Router A is the backup and C the designated router. 
- For Router C, both router A and B is the backup. 

At some point in my testing, the database changed. I am still unaware of why. But previously, both which routers was backup or DR were different. 
If this came to be due to the change in reference-bandwidth for costs, is not clear to me yet. From what I understand previously, the change of priority in each router would be
the change that could manually set which router would have which role - much like we did in a previous exercise. For ospf I find the command 'ip ospf priority x' - although I have not used that on my own network. 

From the list above, I do find it peculiar. In my mind it would be natural that a certain router logically connected would be the designated router. For example - for Router A I would feel its more natural that router B is DR and router C is backup.

The rules I find describe the following:

>1. Router is highest OSPF priority will become DR and router with second-highest priority will become BDR.
>2. If the priority of the routers are same then, the router with the highest Router ID is selected as DR and the router with second-highest Router ID is selected as BDR.

If then looking at the router ID that I set manually earlier, 
Router B with address/ID 10.8.83.254, which in decimal equals to 168317950
Router C with address/ID 10.8.83.255, which in decimal equals to 168317951

According to this, it should be router C that is set as designated router, and B as backup. However - if looking at a traceroute between Lubuntu3 and Lubuntu1, it chose the route straight from B to A.

![](/images/network-doc/E08/routerfromL3toL1.png)

Continuing with Router A, trying to understand the information we have seen above.

```
C>* 10.8.83.0/28 is directly connected, eth0.25, 01:15:23
O   10.8.83.240/30 [110/1] is directly connected, eth1, 01:14:34
C>* 10.8.83.240/30 is directly connected, eth1, 01:15:15
O>* 10.8.83.244/30 [110/2] via 10.8.83.242, eth1, 01:14:32
O   10.8.83.248/30 [110/1] is directly connected, eth2, 01:14:33
C>* 10.8.83.248/30 is directly connected, eth2, 01:15:21
O   10.8.83.253/32 [110/0] is directly connected, lo, 01:15:20
C>* 10.8.83.253/32 is directly connected, lo, 01:15:25
O>* 10.8.83.254/32 [110/1] via 10.8.83.242, eth1, 01:14:32
O>* 10.8.83.255/32 [110/1] via 10.8.83.250, eth2, 01:14:29
O   192.168.39.0/26 [110/10] is directly connected, eth0.5, 01:15:07
C>* 192.168.39.0/26 is directly connected, eth0.5, 01:15:22
O   192.168.39.64/26 [110/10] is directly connected, eth0.15, 01:15:07
C>* 192.168.39.64/26 is directly connected, eth0.15, 01:15:22
O>* 192.168.39.128/26 [110/11] via 10.8.83.242, eth1, 01:14:32
```

C> * ... and C * all represent 'directly connected'. 
In this case we see all the subnets that exists for the router: .0, .240, .248, .253(loopback lo) 
and the subnets to the Lubuntu1 and 2 workstation groups. 

The routes are shown as selected with the marker '>' and either Connected as mentioned above, or O for ospf. 

Further, I believe we can see from '10.8.83.244/30 [110/2] via 10.8.83.242, eth1,'  describes 
the route to subnet .244/30 go via  eth1 / .242 which is the interface on router B that connects to router A. 
From there - packets can move through to subnet .244/30

I could add more traceroutes around, but I feel its working quite well and from what I have seen so far I am satisfied with the result and connectivity as each devices reach around the network. 

As a backup, I save the outputs of ping from Lubuntu2 to various parts of the system:

```
Lubuntu-2: ping 192.168.39.5
PING 192.168.39.5 (192.168.39.5) 56(84) bytes of data.
64 bytes from 192.168.39.5: icmp_seq=1 ttl=63 time=8.89 ms
64 bytes from 192.168.39.5: icmp_seq=2 ttl=63 time=5.34 ms
^C
--- 192.168.39.5 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 5.340/7.112/8.885/1.772 ms
Lubuntu-2: ping 192.168.39.135
PING 192.168.39.135 (192.168.39.135) 56(84) bytes of data.                    
64 bytes from 192.168.39.135: icmp_seq=1 ttl=62 time=6.73 ms                  
64 bytes from 192.168.39.135: icmp_seq=2 ttl=62 time=4.20 ms                  
^C                                                                            
--- 192.168.39.135 ping statistics ---                                        
2 packets transmitted, 2 received, 0% packet loss, time 1002ms                
rtt min/avg/max/mdev = 4.204/5.465/6.727/1.261 ms                             
Lubuntu-2: ping 10.8.83.245
PING 10.8.83.245 (10.8.83.245) 56(84) bytes of data.                          
64 bytes from 10.8.83.245: icmp_seq=1 ttl=63 time=4.06 ms                     
64 bytes from 10.8.83.245: icmp_seq=2 ttl=63 time=5.11 ms                     
^C
--- 10.8.83.245 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 4.061/4.583/5.106/0.522 ms
Lubuntu-2: ^C
Lubuntu-2: ping 10.8.83.250
PING 10.8.83.250 (10.8.83.250) 56(84) bytes of data.
64 bytes from 10.8.83.250: icmp_seq=1 ttl=63 time=2.95 ms
64 bytes from 10.8.83.250: icmp_seq=2 ttl=63 time=1.20 ms
^C
--- 10.8.83.250 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.204/2.076/2.949/0.872 ms
Lubuntu-2: ping 10.8.83.249
PING 10.8.83.249 (10.8.83.249) 56(84) bytes of data.
64 bytes from 10.8.83.249: icmp_seq=1 ttl=64 time=5.89 ms
64 bytes from 10.8.83.249: icmp_seq=2 ttl=64 time=1.00 ms
64 bytes from 10.8.83.249: icmp_seq=3 ttl=64 time=1.44 ms
^C
--- 10.8.83.249 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.003/2.776/5.885/2.205 ms
Lubuntu-2: ping 10.8.83.241
PING 10.8.83.241 (10.8.83.241) 56(84) bytes of data.
64 bytes from 10.8.83.241: icmp_seq=1 ttl=64 time=3.13 ms
64 bytes from 10.8.83.241: icmp_seq=2 ttl=64 time=3.95 ms
^C
--- 10.8.83.241 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 3.129/3.539/3.950/0.410 ms
Lubuntu-2: ping 10.8.83.1
PING 10.8.83.1 (10.8.83.1) 56(84) bytes of data.
64 bytes from 10.8.83.1: icmp_seq=1 ttl=63 time=9.59 ms
64 bytes from 10.8.83.1: icmp_seq=2 ttl=63 time=4.76 ms
^C
--- 10.8.83.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.763/7.177/9.592/2.414 ms
Lubuntu-2: ping 10.8.83.2
PING 10.8.83.2 (10.8.83.2) 56(84) bytes of data.
64 bytes from 10.8.83.2: icmp_seq=1 ttl=63 time=4.63 ms
64 bytes from 10.8.83.2: icmp_seq=2 ttl=63 time=6.83 ms
^C
--- 10.8.83.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 4.629/5.731/6.834/1.102 ms
Lubuntu-2: ping 10.8.83.3
PING 10.8.83.3 (10.8.83.3) 56(84) bytes of data.
64 bytes from 10.8.83.3: icmp_seq=1 ttl=64 time=0.921 ms
64 bytes from 10.8.83.3: icmp_seq=2 ttl=64 time=6.93 ms
^C
--- 10.8.83.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1024ms
rtt min/avg/max/mdev = 0.921/3.924/6.928/3.003 ms
Lubuntu-2: ping 10.8.83.253
PING 10.8.83.253 (10.8.83.253) 56(84) bytes of data.
64 bytes from 10.8.83.253: icmp_seq=1 ttl=64 time=2.43 ms
64 bytes from 10.8.83.253: icmp_seq=2 ttl=64 time=1.22 ms
^C
--- 10.8.83.253 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.220/1.823/2.427/0.603 ms
Lubuntu-2: ping 10.8.83.255
PING 10.8.83.255 (10.8.83.255) 56(84) bytes of data.
64 bytes from 10.8.83.255: icmp_seq=1 ttl=63 time=1.32 ms
64 bytes from 10.8.83.255: icmp_seq=2 ttl=63 time=1.43 ms
^C
--- 10.8.83.255 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.317/1.371/1.425/0.054 ms
```


- [x] Connectivity tests



# OSPF router security

# Passive interfaces
While adding the passive-interfaces I found that I had an error in my physical topology. The logical was correct - however the physical showed the wrong interfaces. 
I think it is due to the design of my routers where in one A and B are on opposite sides than in the other schematic. After correcting that, I added eth2 on Router B as 
a passive interface, while having my lubuntu wireshark listening on the physical line LAN4 between the parties mentioned. The **gif** below shows the process.

![](/images/network-doc/E08/Disablingeth2RouterB.gif)

After rebooting the router, I do see one hello being registred in wireshark. By looking at the information though, I see that the source IP is router B, but the designated router 
IP which previously was 192.168.39.129 - aka eth2, now shows 0.0.0.0 and there are no more packages registred once time goes by.

The two other interfaces on Router B goes to the other routers, and the same goes for Router C. However for Router A - I do have an interface that currently is sending hello packets down 
to Network_Switch_B -> and further to the other switches and workgroups connected there. There is no reason to send hellos out that interface, because there is no way we can form an adjacency to the devices from that interface and onwards. 

Due to this, I also set eth0 (between router A and Switch B) as passive-interface, in addition to also 0.5, 0.15 and 0.25 which are the VLANS we created in an previous exercise. 

![](/images/network-doc/E08/passiveinterfacesrouterab.png)


# MD5 key authentication between routers
I set one key on the interfaces between router A and B, and another key between the interfaces connecting Routers A and B to router C. I try to add two keys, to keep them separate. 
![](/images/network-doc/E08/md5keysconfig.png)


I set the Lubuntu-wireshark VM to listen on the line "RoAToRoB" between Router A and B. 

From the information gained using wireshark, I see new lines within the OSPF fields describing authentication. These update on each hello-packet, and I assume they have the MD5 encryption and not just a conversion. Its been a while since I played around with hashvalues.
 I start an attempt to test them with a program called john, vs an example password file i wrote quickly, but initially it fails. Initially I find it hard to use the file gained from the ettercap print, and I notice I did the cut wrong so the values arent easily identified. 
 
 
![](/images/network-doc/E08/wireshark-key.png)

![](/images/network-doc/E08/Collectinghashes.png)

Instead I copy one of the hashes straight from Wireshark and save that as a textfile routeratobsinglehash.txt, of which the program hashid gives some alternatives. Although since I already know it is MD5, I try that format in John and come up with a couple pretty decent hits from my password list. Its missing a digit in the end, but seeing atleast that the name appeared it seems the MD5 authentication works. 
![](/images/network-doc/E08/Collectinghashes2.png)


- [x] Configure cybersecurity to OSPF







```
Notes:
- Last week "switching loops" in OSI L2. Also a phenomenon called "routing loops" on L3. TTL-field in L3-packets helps preventing meltdowns.
- Static routing vs dynamic routing. Instead of creating routes, the router devices broadcast information and listen to information between eachother "adjacencies". 
- Large scale, distance vector. Distance/direction. Main protocol in this course is called Border Gateway Protocol.
- OSPF Open Shortest Path First. Calculating cost of each route by measuring/taking the metric(calculations) based on links between devices, cable quality, distance and such. 
Djikstra's algoritmn, which tell the shortest path from _one_ node to every other node. 

- Changing bandwidth reference must be done on all routers. This seems to be quite vital in particularily fast ethernets. By default the reference-bandwidth is 100 Mbits/sec. 
If you then have a 100 Mbits line, it has a cost of 1. Seeing then that you also have a line of 10 Gbit, the cost-value can not be lower than 1, hence it gets registred with the same cost. 
To take advantage of faster speeds, change the reference value. 

Adjacencies between routers are formed through exchange of OSPF -packets. 

Router-ID is the IPv4 address, set by the admin. This so the routers will know who is where/what. If this is not done/forgotten - default is that the highest binary value of the IPv4 addresses of loopback interfaces. 
Lastly if this is not avaliable, the highest IPv4 of other interfaces will be the ID. 
```

![](/images/network-doc/E08/RouterRelationship_NetworkTypesOSPFv2.png)

![](/images/network-doc/E08/OSPFNeighborshipStates.png)

- show ip route

C, physically connected, routes are better than dynamically learnt routes. 

- show ip ospf

instability? check SPF run times. If instability, it updates and runs many times. 
