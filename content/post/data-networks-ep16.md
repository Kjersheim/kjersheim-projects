---
title: "Data Networks Episode 16: IPv6 – Making Things Dual-Stack"
date: 2023-04-22
draft: false
summary: "This episode introduces IPv6 integration into an existing network topology, covering IPv6 subnet planning, manual address configuration on Lubuntu and VyOS devices, enabling dynamic routing with OSPFv3, and verifying connectivity using ping, traceroute, and web/SSH access."
---

# Plan

- [ ] Form IPv6 addresses
- [ ] Assign addresses to topology
- [ ] Manually add IPv6 addresses
	- [ ] Lubuntus
	- [ ] Routers

- [ ] Add dynamic routing, OSPFv3
- [ ] Verification and connectivity

# Forming IPv6 addresses

*Student no. 2208339  // vvxyzki*

From the above number I formt he address 2001:2208:339:<subnets>::/64, Which can described as:

>![](/images/network-doc/E16/IPv6address.png)

This means I have 2001:2208:339:0::/64 to 2001:2208:339:ffff::/64 avaliable for my network within this main subnet, 2001:2208:339::/48.

As the standard suggested size of a subnet is /64 on IPv6, I plan that size on the workstation subnets around the network. When it comes 
to the network devices, as mentioned in the materials there is an ongoing discussion weather or not to use minimal sized subnets, which would be /127. 

From my understanding, there are plenty of addreses, and despite feeling wasteful it might be fine to select /64 also for my network devices. As stated on multiple forums
and portals such as StackExchange, /127 is possible with a P2P network where there will ever only be 2 hosts, the two peers, since there are no broadcast addresses and such like
we have in IPv4. 

However, as some vendor equipment seems to in some instances not allow that size. Using a /127 size address may be harmful if the subnet router is has an anycast address implemented.

Althoguh, being able to quickly identify or visualize the type of address is favourable in my mind, so I chose to set the subnet size for network devices to /126. 
The exception will be the subnet giving addresses to the switches, which I want to be able to expand without changing major parts of the configuration, so I will set that to the standard /64.
Loopback addresses will be set to /128, and using the same ID as the IPv4 equivalent for convenience sake.

- [x] Form IPv6 addresses

>![](/images/network-doc/E16/E16IPv6PlanningChartV4.png)

# Logical topology

Incorporating the above plans into the logical chart

>![](/images/network-doc/E16/E16NetworkCharts-LogicalTopologyV4.png)

- [x] Assign addresses to topology

# Assigning IPv6 addresses

## Router A
```	
set interfaces ethernet eth1 address chosen-ipv6-address
set interfaces ethernet eth0 vif x address chosen-ipv6-address
```	

>![](/images/network-doc/E16/RouterAIPv6.png)

### show ipv6 route
>![](/images/network-doc/E16/RouterAShowIpv6Route.png)

The route type, which can be one of the following:

C - The destination is directly connected to the router.
S - The route is a static route. (Currently no static routes)
O - The route is learned from OSPFv3.



## Router B

>![](/images/network-doc/E16/RouterBIPv6.png)

### show ipv6 route

>![](/images/network-doc/E16/RouterBShowIpv6Route.png)

## Router C

>![](/images/network-doc/E16/RouterCIPv6.png)

### show ipv6 route

>![](/images/network-doc/E16/RouterCShowIpv6Route.png)

### Lubuntus

Adding the IPv6 manually based on the topology chart. 

- [x] Manually add IPv6 addresses
	- [x] Lubuntus
	- [x] Routers



## Enabling OSPFv3

On all 3 routers, using the IPv4 version of the routers loopback addresses as a router-id.

```	
set protocols ospfv4 parameters router-id <ipv4 lo address>
```	

Furthermore, adding each interface to area 0.0.0.0 using

```	
set protocols ospfv3 area 0.0.0.0 interface <interface>
```	

Output of the show ipv6 command linked above. 

- [x] Add dynamic routing, OSPFv3


# Verification and connectivity

On Lubuntu1 using the command ip addr shows the ipv6 is added:

>![](/images/network-doc/E16/Lubuntu1IP.png)

From Lubuntu1 I ping the other machines, including LubuntuWAN. 

>![](/images/network-doc/E16/Lubuntun1ToAllLubuntus.png)

LubuntuWAN pinging all routers, and loopback of Router C:

>![](/images/network-doc/E16/LubuntuWANRouters.png)

>![](/images/network-doc/E16/LubuntuWANIPAndRouterC.png)

Traceroute from LubuntuWAN to Lubuntu1:

>![](/images/network-doc/E16/TRacerouteLubuntuWANLubuntu1.png)

>![](/images/network-doc/E16/TracerouteLubuntu1To2.png)


Adding IPv6 address with name lubuntu1.ip6 in the /etc/hosts file on LubuntuWAN, testing the address from firefox:

>![](/images/network-doc/E16/httpfromlubuntuwan.png)

>![](/images/network-doc/E16/etchostslubuntuWAN.png)

>![](/images/network-doc/E16/httpname.png)

SSH connection between Lubuntu2 and Router A:

>![](/images/network-doc/E16/sshroutera.png)

- [x] Verification and connectivity


## Updated configuration-files after completing E16

[Network_Switch_A](/images/network-doc/E16/Config_files/E16-SwitchAu.cfg)\
[Network_Switch_B](/images/network-doc/E16/Config_files/E16-SwitchBu.cfg)\
[Network_Switch_C](/images/network-doc/E16/Config_files/E16-SwitchCu.cfg)

[Vyos_Router_A](/images/network-doc/E16/Config_files/E16-RouterAu.cfg) \
[Vyos_Router_B](/images/network-doc/E16/Config_files/E16-RouterBu.cfg) \
[Vyos_Router_C](/images/network-doc/E16/Config_files/E16-RouterCu.cfg) 

- [x] Update configuration files with E15 and 16 changes



## Notes
```	
- Can be shortened once with a double colon:
2010:2222:0000:0005:0000:0000:0000:0002 to 2010:2222:0000:0005::2

0's can be omitted, but - if the whole 4 hex field is 0000 it needs to be added as 0 total value.

```	

