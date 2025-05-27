---
title: "Data Networks Episode 3: Ethernet, Switching and VLANs"
date: 2023-02-02
draft: false
summary: "Subnetting exercises, registry lookups, and practical analysis of IP address ranges using real-world tools and registries."
---

Reference number used to change subnet values: 2208339  // vvxyzki

10.xy.yz.0/24     ->     10.08.83.0/24  
192.168.ki.0/24   ->     192.168.39.0/24

# Calculating Networks

### Exercise 1
```
192.168.39.0/24
```
*Address range:*  
192.168.39.0 **/24** – 8 bits remaining, 2^8 = 256 possible addresses in the range 0 - 255.

...

*Broadcast address:*  
192.168.39.255

*Subnet address:*  
192.168.39.0

...

| Specification | Subnet No. 1 | Subnet No. 2 | Subnet No. 3 | Subnet No. 4 |
|---------------|--------------|--------------|--------------|--------------|
| Network address  | 192.168.39.0/26 | 192.168.39.64/26 | 192.168.39.128/26 | 192.168.39.192/26 |
| New subnet mask  | 255.255.255.192 |
| Number of usable host addresses | 62 | 62 | 62 | 62 |
| First IP Host    | 192.168.39.1 | 192.168.39.65 | 192.168.39.129 | 192.168.39.193 |
| Last IP Host     | 192.168.39.62 | 192.168.39.126 | 192.168.39.190 | 192.168.39.254 |
| Broadcast address| 192.168.39.63 | 192.168.39.127 | 192.168.39.191 | 192.168.39.255 |

...

### Exercise 5

**Is 10.08.83.0/24 a part of 10.08.16.0/20?**

Comparing binary:
![](/images/network-doc/E03/CompareBinaries.png)

**Answer:** ❌ No, it is not within the supernet.

...

### Looking up addresses

IPv6 shown by default:
![](/images/network-doc/E03/IPv6.png)

IPv6 results:
![](/images/network-doc/E03/RipeResultIPv6.png)

Disabled IPv6:
![](/images/network-doc/E03/DisablingIPv6.png)  
![](/images/network-doc/E03/IPv4.png)  
![](/images/network-doc/E03/RipeResultIPv4.png)

...

# Internet Registries

**195.08.83.0/24**
---
![](/images/network-doc/E03/address1_apnic.png)  
![](/images/network-doc/E03/address1_apnic2.png)  
![](/images/network-doc/E03/address1_arin.png)  
![](/images/network-doc/E03/address1_arin2.png)  
---

**43.08.39.0/24**
---
![](/images/network-doc/E03/address2_milacnic.png)  
![](/images/network-doc/E03/address2_milacnic2.png)  
![](/images/network-doc/E03/address2_arin.png)  
---

**15.39.08.0/24**
---
![](/images/network-doc/E03/address3_arin.png)  
---

**100.83.39.0/24**
---
![](/images/network-doc/E03/address4_arin.png)  
![](/images/network-doc/E03/milacnic.png)  
---
