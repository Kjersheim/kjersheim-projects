---
title: "Data Networks Episode 12: Controlling Network Configuration"
date: 2023-03-25
draft: false
summary: "Overview of the current network configuration files for routers and switches. Introduction to web interfaces for managing network devices, including GUI access and limitations for Vyos."
---

# Controlling Network configuration

>This page covers the network configuration files of all network devices so far in the project. Additionally web-interfaces to configurate remotely will briefly be introduced. 


## Current configuration files (Updated with E12 for Switch A and Router A)

[Network_Switch_A](/images/network-doc/E12/Config_files/E12-SwitchAu.cfg)\
[Network_Switch_B](/images/network-doc/E12/Config_files/E12-SwitchB.cfg)\
[Network_Switch_C](/images/network-doc/E12/Config_files/E12-SwitchC.cfg)

[Vyos_Router_A](/images/network-doc/E12/Config_files/E12-RouterAu.cfg) \
[Vyos_Router_B](/images/network-doc/E12/Config_files/E12-RouterB.cfg) \
[Vyos_Router_C](/images/network-doc/E12/Config_files/E12-RouterC.cfg) 

## Enabling web interface on Router A and Switch A

>![](/images/network-doc/E12/enablehttpsroutera.png)

>![](/images/network-doc/E12/enablehttpsswitcha.png)


### Network Switch A 

>![](/images/network-doc/E12/SwitchAWebinterface.png){width=60%}

![](/images/network-doc/E12/SwitchConfigurationMenu.png) 
 
 From the top menu there are various options for \
 monitoring events errors and such, links to documentation \
 and also applications like the CLI and file manager. 

The configuration-part of the menu shows ability to do a quick-setup for the switch itself

![](/images/network-doc/E12/Switchquicksetup.png)

Further on the menu I have access to the different ports enabled on the devices, where I have the general information about each port. There is also an ability to enable/disable the listed ports. 

For VLANs there are similar abilities as for ports, information and ability to add information, add ports, or delete the VLAN. 

Beyond that there is also ability to
- have and edit Access Control Lists/Policies
- see information about useraccounts on the switch, with possibilities to edit these and add further security options

![](/images/network-doc/E12/securityoptions.png){width=20%}

- Audio Video Bridgin, which seem to split traffic on the network into two groups; real-time traffic and the rest
- Chalet settings, which seems to be the name of the web gui program. 


### Vyos Router A

>![](/images/network-doc/E12/vyosgui.png){width=60%}

Initially by following the materials in this section I realize that it also has the same no-content page as I get when I load up the address as seen above. From some quick searches I find that the official vyos web-gui is still under development, though that there are
various API and applications. So I am at this point not able to control the router via a web interface - without adding another program like VyControl. As the assignment is not involving these applications further I continue to E13. 