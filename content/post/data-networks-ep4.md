---
title: "Data Networks Episode 4: IPv4 addresses, subnetting and ARP"
date: 2023-02-08
draft: false
summary: "Adding a router VM, building logical and physical topologies, and connecting Lubuntu VMs via VLANs and ARP using VyOS."
---

## Plan
- [x] Add router VM  
- [x] Topology charts  
- [x] Configure switches (A & B + Management VLAN)  
- [x] Configure router  
- [x] Configure Lubuntu1 and 2  

---

**Starting the exercise with importing the router-image as a new VM, named Vyos 1.**  
![](/images/network-doc/E04/FirstRouterImport.png)

Adding a network adapter, nr. 4, to Network_Switch_B, and setting it to an internal network I named RoTOSw (Router to Switch).  
![](/images/network-doc/E04/Router1Adapter1.png)  
![](/images/network-doc/E04/SwitchBNewAdapter1.png)

Making sure to keep the chart with the physical topology updated:  
![](/images/network-doc/E04/PhysTop_Router1.png)

---

## Addresses to use

Using 2 subnets of /26 (62 usable IPs) and 1 of /28 (14 usable IPs):  
![](/images/network-doc/E04/ChosingSubnets.png)

```
192.168.39.0/26
192.168.39.64/26
10.08.83.0/28
```

Creating charts for Layer 1–3 (Physical to Network in OSI model):  
![](/images/network-doc/E04/Layer123OSI.png)  
![](/images/network-doc/E04/LogicalChart_1.png)

---

# Implementing configurations

## Configuring Switches

- VLAN "secretbasement" (tag 15) added to both switches.
- VLAN "workstations" (tag 5) was already in place.
- Tagged/untagged ports adjusted for VLANs according to logical layout.

### Network_Switch_B  
![](/images/network-doc/E04/ConfigSwithcB_1.png)

### Network_Switch_A  
![](/images/network-doc/E04/ConfigSwithcA_1.png)

Default gateway added:  
![](/images/network-doc/E04/SwitchDefaultGateway.png)

---

## Configuring Router

⚠️ Note to self: Always shut down VyOS VMs with `poweroff`.

Checking adapter MACs and interfaces:  
![](/images/network-doc/E04/vyosadapter1mac.png)  
![](/images/network-doc/E04/VyosSettingsAdapter1Mac.png)

Setting IPs for VLANs:  
![](/images/network-doc/E04/Vyos1configuration_1.png)

---

## Configuring Lubuntu machines

### Lubuntu 1  
![](/images/network-doc/E04/Lubuntu1Config_1.png)  
Ping to router via VLAN "workstations":  
![](/images/network-doc/E04/Lubuntu1_Config_ping.png)

### Lubuntu 2  
![](/images/network-doc/E04/Lubuntu2Config.png)

✅ Successful connections from both Lubuntu clients to router.

---

## Management VLAN

- Created VLAN `network_devices` with tag 25.
- Tagged inter-switch links and router interface.
- Removed old IPs from switches and added new logical IPs.

Final switch VLAN config:  
![](/images/network-doc/E04/SwitchA_B_networkdevicesconfig.png)

### Troubleshooting:
Discovered Adapter 1 on Vyos was connected to wrong network (LAN1 instead of RoTOSw). After reassigning, the ping started working.

New router:
- Name: Vyos Router A  
- MAC: 08002793E377

New interfaces and VLANs added successfully:  
![](/images/network-doc/E04/NewRouterInterfaces.png)

---

## Updated Topologies

### Physical topology  
![](/images/network-doc/E04/E04NetworkChart_04-Physical_Topology.drawio.png)

### Logical topology  
![](/images/network-doc/E04/E04NetworkChart_04-Logical_Topology.drawio.png)

---

# Connectivity Tests

### Lubuntu <-> Lubuntu  
![](/images/network-doc/E04/ConnectLubuntus.png)  
Traceroute:  
![](/images/network-doc/E04/ConnectLubuntusTraceroute.png)

### Lubuntu <-> Router  
![](/images/network-doc/E04/Connect2LubuntusToRouter.png)  
![](/images/network-doc/E04/ConnectTracerouteLubuntuToRouter.png)

### Lubuntu <-> Switches  
![](/images/network-doc/E04/ConnectLubuntusToSwitches.png)  
Traceroute:  
![](/images/network-doc/E04/ConnectLubuntuToSwitchesTraceroute.png)

---

# Configuration Files

Copied via SSH:  
![](/images/network-doc/E04/SSH.png)

- [Network_Switch_A](/images/network-doc/E04/Network_Switch_A.cfg)
- [Network_Switch_B](/images/network-doc/E04/Network_Switch_B.cfg)
- [Vyos Router A](/images/network-doc/E04/VyosRouterA.cfg)
