---
title: "Data Networks Episode 10: IPv4 Network Address Translation"
date: 2023-03-12
draft: false
summary: "Setting up and verifying many-to-one NAT on VyOS, reflecting on address translation concepts, and analyzing private-to-public traffic flow using Wireshark."
---

### Initial reflection
```
Going through the first materials on the subject discussing NAT. When being introduced with this, I immidiately was thinking that would it not just be as if the traffic was routed to the gateway/router, and that an interface on the router
was connected to the ISP subnet aka. connected to the open internet. Further on - why does it need to be translated? Does not the interface have its own address anyway? Could the router not store the information that each user
on my subnet was requesting through the router and the internet? 

On that note, I remembered an old issue dealing with ISP IP and phone logs. It is about 10 years since I first heard the term "NAT". Back at the time, we were already investigating various IP-addresses on a daily basis. By speaking to contacts within the ISPs, 
I was told that it was impossible to identify the user of a certain IP-address, due to the address being a NAT-address. When trying to get an explanation about this - seeing I had never had any experience with networking, I was told that due to the lack of IPv4 addresses, 
the ISP made the users share the same. This specially occured on mobile-devices at the time. The ISP/operator would have a server translating the addresses and having x amount of devices connected to it. When device x.1 wanted access to the internet, it gave the devices an address. 
Shortly after, device x.1 did no longer need access, but device x.2 did. The server switched the addresses so x.2 would now have the address x.1 had a short second ago. So within a few minutes, several devices would have used the same IP. 

If we imagine that the different devices x on the mobile basestation is the devices in our home network, each device will send information to the router, and the router would use the same address to reach outside our home network to the internet with the different sources in our home network. 

(On a sidenote: we did eventually realize that the ISP according to the laws in Norway also would need to have logs of this exchange of addresses and the subscribers, since they charged their subscribers money. So after discussing this with the ISP/Operator, we were often given
the logs and then tactically filtered the users of the IP based on other kinds of information)
```

## Plan

- [ ] Delete firewall configured in E09
- [ ] Configuring NAT
	- [ ] Configure Many-to-1
- [ ] Investigating traffic
	- [ ] CLI - commands
	- [ ] Wireshark


# Firewall removal

I go through my router 3 which was where the firewall was located. I remove all rulesets, and all policy-zones.

> delete firewall name LAN_to_WAN

Then repeating the above for all firewall names, LAN_to_vyos, vyos_to_LAN etc. After that I deleted all the zones.

> delete zone-policy zone LAN   (and WAN, then vyos)

- [x] Delete firewall configured in E09


# Configuring NAT

## Testing pre NAT

For the purpose of the exercise I clone the old Lubuntu Wireshark VM, calling the clone Lubuntu Wireshark 2. I position the Wireshark1 between Router A and C. I position Wireshark2 between Router C and LubuntuWAN.

I proceed to start a ping from Lubuntu 1 to Lubuntu WAN, and capture the packets on both sides of Router C. 

![](/images/network-doc/E10/PingLub1toLubWANBEFORE.png)

As you see in the picture the source and destination is visible from both sides, marked with red and black squares in the picture. 



From the topology-charts I see that the interface on router C which will be handling outbound traffic is eth2:

>![](/images/network-doc/E10/outboundinterface.png)

>![](/images/network-doc/E10/settingoutboundinterface.png)

>![](/images/network-doc/E10/outboundinterfacewireshark.png)

Already now, after masquerading the traffic, we see that the information that is being sent out is shown as 43.5.39.1 - which is interface eth2 on Router C - when in reality it is a ICMP request from Lubuntu1 (192.168.39.5).
When going deeper in the Wireshark readings, I notice that both the IP information about packets and the frame information is showing the ip and mac of the router C. Nowhere in the fields am I able to find any private addresses of my network. 

Since I deleted my firewall, I am now able to ping from Lubuntu WAN inside my LAN. I ping Lubuntu1 from WAN, and now I do see information about both original source and destination, as seen in the picture below.

>![](/images/network-doc/E10/WAntoLubuntuBefore.png)

I perform the same tests between Lubuntu2 and Lubuntu WAN, with the same results. 

At this point I am wondering the purpose of adding each subnet.. Then I do some more testing. While I add the subnet that contains Lubuntu1 to the NAT, I simultaneously do tests. Nothing has changed when I ping from Lubuntu1 to Lubuntu WAN. 

**However** at this point I also keep pinging between Lubuntu2 and Lubuntu WAN, after adding the subnet of Lubuntu1 to the NAT. At this point suddenly I start seeing the source directly back to Lubuntu2! 

>![](/images/network-doc/E10/Lub2toWAn1.png)

>My conclusion so far is that if we add outbound interface and enable masquerading, all traffic is masqueraded to look like the outbound interface address. If we add a subnet as source address, only those added are masqueraded. 

I add the rest of the subnets, so they will also be accounted for in the NAT. After adding Lubuntu 2s subnet to the NAT it also now hides the original source and refers to the routers interface address/IP. After doing this, I notice
a flaw in my theory. Only the last added address seem to persist in the nat. So rather than adding all to one rule, perhabs we need to add a rule per subnet? At this stage the words of "subnet calculations" come into my mind. 

> New conclusion, I need to increase the size of the subnet to be added as the source, to cover all 3 lubuntu machines(or my "different workstation subnets", but which have the same format in addresses).

## Finding the subnets to add to NAT

The subnet 192.168.39.0/26, .64/26 and .128/26 are all used by my three workstation-areas. I need to add all of these within the translation, as I do not want their information to reach outside of my LAN. 

From previous we know that /26 leaves 6 bits for the host addresses, 64 minus 2(subnet and broadcast) hosts. We have three of these subnets, so we need to increase the size. If we go up to 7 bits its still only room for 128, so we need
to increase the host addresses to 8 bits, 2^8 = 256 addresses. So the address to be added would be 192.168.39.0/24.

Doing the same tests over again with a ping from both Lubuntu1 and Lubuntu2 to Lubuntu WAN now both hide the source IP and shows the router C outbound interface. 

>![](/images/network-doc/E10/Lunb1and2PingLubWANafter.png)

At this point it seems to translate the source from within my network to the outside from my workstations. I do however also want to hide my network devices. 
By going over the subnets for these network devices, I see that there are 3 subnets sized /28, 3 subnets sized /30 and also the 3 loopback addresses. 
The 4 bit addresses have then 2^4=16 hosts, while the 2 bit addresses have 4 hosts. The loopback addresses are 1 each. In total then I need 16+16+16+4+4+4+3 = 63. 

By this I would need a subnet size of atleast 6 bits for the host, aka. /26. However, despite having room to grow within the subnets themselves, 
I still would like some more room on the outside for future possibilities without having to change the NAT configuration. /25 would be more than enough with 128 hosts, but 
for good measure I add /24 so it has 256. 

>![](/images/network-doc/E10/addingrule20.png)


### Router C show config - NAT

>![](/images/network-doc/E10/R3NATconfig.png)

- [x] Configuring NAT
	- [x] Configure Many-to-1
	
	

## Verfifying functionality

In the above testing while adding the different subnets and which ones to add, I have already tested with wireshark on both traffic from Router C and into my network, aswell as from Router C and outwards. As a last verification I repeat as seen below.

Ping from Lubuntu1 to Lubuntu WAN, private to public:

>![](/images/network-doc/E10/PingPrivatetoPublic.png)

From the wireshark capture I see that the sniffer inside the network can see the source and destinations, however the sniffer placed outside the Router C can only see the .1 address which is the routers outbound interface. 

To test the wiresharks also on the network devices, I perform a ping from Switch A to Lubuntu WAN: 

>![](/images/network-doc/E10/networkdevicestopublicping.png)

From these tests it seems also the traffic from the network devices are, as intended, translated.

- [x] Investigating traffic
	- [x] CLI - commands
	- [x] Wireshark