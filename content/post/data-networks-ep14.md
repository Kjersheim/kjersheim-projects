---
title: "Data Networks Episode 14: WLAN Home Network Analysis"
date: 2023-04-10
draft: false
summary: "A comprehensive wireless network survey using Windows 10 tools and third-party applications like WiFi Analyzer. Includes analysis of 2.4 GHz and 5 GHz channel utilization, router configuration, and implications for performance and interference."
---

## What can we see using the operating system Windows 10?

From the lower right menu the wireless local area networks in range of my laptop are listed. Without being connected to one, there is not too much information to be gained by using the standard OS information lookup tools. 

>![](/images/network-doc/E14/wifi1.jpeg)

As seen it is connected to the network called "DNA-WIFI-48D4". \
*On a sitenote: We have not changed anything within our home wireless network, besides putting a more secure password. The names and settings are set by the operator DNA. Some settings, like the public IP, will be anonymised*

**Disclaimer:** For the purpose of this section, I will be using a laptop and not the stationary machine hosting the VMs used so far in the project. 


## WiFi Analyzer

This is a program in the windows application store, made by Matt Hafner. It is used to see the channels of WiFi networks in our surrounding areas, troubleshoot WiFi problems and the best locations for our routers or access points. 
The tool will be used to go through the networks in the building I reside in. 


Loading up the application will immidiately show alot more information than we initially saw in the list. 

>![](/images/network-doc/E14/wifi2c.jpg)

In the top we see the dBm, which lists how powerful the signal strenght and how well I am connected to my WiFi network. 
Further down we see the name of the wireless (SSID) which channel it is currently using and also the frequency my wireless adapter is operating on, and how its connected to the wireless network. 

We can see the bandwidth and protocol. The protocol is important to see another important point. It shows if its a b,a, g, n, or ax and so on. This matters, as it identifies and determines how well I am connected 
to the wireless network. 

>![](/images/network-doc/E14/wifiprotocol.png){width=10%}

As we continue down the list it shows our private IPv6 address within our home network, as well as our IPv4 public address from the router and to the internet. 
Also shown is the authentication type and encryption, as well as network uptime.

If we change view to "Networks" we see our current connected network on the top, and other networks in range in a list below, where I can see the other networks SSIDs, channel, signal strenght and if they are in or out of range. 

>![](/images/network-doc/E14/wifi3.jpeg)

Comparing this with the output of the command ipconfig /all and sectioning out the Wifi:

>![](/images/network-doc/E14/wifi4.jpeg)


Interestingly, we see the specifications of the network card, which seems to have a later capacity than we saw in the WiFi Analyzer screen (802.11ax vs 802.11n). The differences from these I read to be Wi-Fi4 and Wi-Fi6, so perhabs there are some upgrades avaliable that I need to identify, or maybe it requires both depending on the connection. 
We see the public and private ip-addresses matches and information about the gateway (.1.1 - the routers private address). There are several temporary addresses, which from my understand is made by services of Windows for security reasons when performing certain tasks, as they do not want to share the preferred address. 


## Channel usage

From the top view of the application we can also go into the analyze section, where the channels are visualized. 


### 2.4 GHz

As we see from the picture below, we have our wireless marked with the wifi-symbol and set on Channel 1. We see two other networks on the same channel, and several others on channel 6 and 11. 
>![](/images/network-doc/E14/wifichannel1.jpeg)

2.4 Channels overlap. Each channel is 20 MHz wide, but the center of the channel is where it is marked and it is only separated by 5 MHz. So if we look at Channel 1 and 2, we see that they overlap by 75%. The only channels that do not overlap is Channel 1, 6 and 11. 

As we see in the picture above, this is most likely accounted for by the router software in the building. It seems most residents are using DNA routers judging by the SSIDs. Most networks seem to want to stay on those frequencies that overlap the least.

Doing some digging on this part gives a visualization that I wanted to add in the project:
>![](/images/network-doc/E14/metageek.jpeg)\
*Source: metageek, https://www.metageek.com*

If overlapping Wi-Fi channels are relatively loud to each other (-75db or above after filtering), they take turns but can't talk to each other. The Wi-Fi protocol won't allow you to start transmitting 
if they can hear enough noise. This is to interwork with other protocols like Bluetooth. Mixture of 20/40/80 MHz overlapping Wi-Fi channels work the same way. If channels overlap but are sufficiently distant, 
Wi-Fi is allowed to transmit. But co-channel interference, even at very great distance, causes transmitters to take turns, which is a bug in the protocol. To avoid taking turns, it's best to use different channels if others are already in use.

Co-channel interference occurs when multiple wireless access points or routers transmit on the same channel in close proximity to each other. When this happens, the signals can overlap and interfere with each other, 
causing a decrease in wireless network performance and speed. The interference is caused by the fact that the Wi-Fi protocol requires devices on the same channel to take turns transmitting, leading to delays and congestion. 
This can be particularly problematic in areas with high population density or in buildings with many walls and obstacles, as signals can reflect and bounce around, leading to more interference. 

To minimize co-channel interference, I find it is recommended to use non-overlapping Wi-Fi channels and adjust the signal strength and placement of access points or routers. Based on the chosen channels though, 
it seems this might be done already by the router either atuomatically or statically from the initial configuration. 


### 5 GHz

Changing my network to the routers 5GHz alternative wireless, and also chosing that on the WiFi Analyzer application ![](/images/network-doc/E14/wifi5ghz.jpeg)

![](/images/network-doc/E14/wifi5ghz2.jpeg)

The distance and options is much greater in the 5GHz frequency. As stated in the materials a 5 GHz channel also has a varying size to increase the bandwidth. The channels are still 20 MHz wide, but they are further apart so they dont overlap. 

Chosing the frequencies will depend on the intended usage and type of device. As I can see (also shown below) most of our devices are utilizing the 5 GHz WiFi, while my laptop is lonely in the list of 2.4 GHz devices. 2.4 has a wider range and 
better at penetrating walls, however its also more prone to overlap other networks. 

On the other hand, 5GHz has a shorter range but offers faster speed and less interference from other devices. Its better for smaller spaces and fewer obstacles. It is also less prone to interference from neighboring WiFi networks using the same channel. 
Considering the house we live in has thick concrete walls, the 2.4 might be a good option for a laptop that can be brought to different rooms. In addition,  the 5GHz network seems quite crowded with all other devices such as mobile phones, game consoles, tv and such. 

Newer devices do however also have dual-band, to switch automatically between frequencies to avoid these types of congestation. 


## Router information

From the previous investigation we find that the gateway/router has the IP 192.168.1.1. Usually these will have, like we explored with the vyos router, web interfaces to change information. Navigating with a browser to the address using HTTP shows the following login screen:

![](/images/network-doc/E14/routerlogin.png)

From the main hub we have access to amongst other information the WiFi section, showing several devices connected to the different frequencies the router provides, as well as information about guestnetworks and such. 

![](/images/network-doc/E14/wifioverview.png)

Furthermore there are many settings that we can adjust and enable/disable. Amongst some MAC filtering, scheduling when to have the wifi avaliable, general information about the wifi such as SSID password and encryption type and more. 

![](/images/network-doc/E14/wifioverview2.png)

Going deeper into the WiFi module we see that the channel selection is automatic, but that it can be set manually. Based on the information discovered above, I will leave it to automatic as it seems to select the best options automatically, and other routers in the building probably do the same. 

![](/images/network-doc/E14/wifichannelrouter.png)

I do also notice the option to change the different WiFi frequencies protocols, but leave them be as they are both having the newest and most containing selected.

![](/images/network-doc/E14/anac.png)

Switching over to the general network settings, we see information about addresses, MAC and IP, for the router, and other such as the DNS. 

![](/images/network-doc/E14/GeneralSettings.png)

Further down we also see the same information about the wireless, as discussed above. 

Going into the various settings we can see statistics and logs, DHCP leases, ARP - which I have not seen before, IPv4 and 6 settings and more. In the IPv4 settings we have the option to change the private IP addresses, the range within the chosen DHCP addresses and so on. 

![](/images/network-doc/E14/ipv4dhcp.png)


