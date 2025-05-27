---
title: "Data Networks Episode 5: DHCP and Static Routing"
date: 2023-02-12
draft: false
summary: "Extending the network with DHCP, static routing, and an additional router and Lubuntu client."
---

## Plan

- [x] Add additional lubuntu and router VM  
- [x] Plan addresses and update charts  
- [x] Configure Router A and B  
- [x] Configure Lubuntu3  
- [x] Configure routers routing table  
- [x] Configure routers to be DHCP servers  
- [x] Change IP configuration to DHCP on all lubuntus  
- [x] Topology charts  
- [x] Connectivity tests  
- [x] Configuration files

---

## Chosing subnets

- `192.168.39.128/26` → for Lubuntu3 and RouterB  
- `10.8.83.240/30` → for RouterA and RouterB  

### Physical topology draft  
![](/images/network-doc/E05/CutoutNewAdditionsPhysical.png)

### Logical topology draft  
![](/images/network-doc/E05/CutoutNewAdditionsLogical.png)

---

## Configuring new devices

### Vyos Router A  
![](/images/network-doc/E05/RouterNameChange.png)  
![](/images/network-doc/E05/RouterAeth1.png)  
![](/images/network-doc/E05/RouterASettingsNewAdapter.png)  
![](/images/network-doc/E05/RouterAeth1Address.png)

### Vyos Router B  
![](/images/network-doc/E05/RouterBInterfaces.png)  
![](/images/network-doc/E05/RouterBAdapter1.png)  
![](/images/network-doc/E05/RouterBAdapter2.png)  
![](/images/network-doc/E05/InconsistentInterfacesRouters.png)  
![](/images/network-doc/E05/RouterBeth1_2_ip.png)  
![](/images/network-doc/E05/CurrentLogicalCutout2.png)  
![](/images/network-doc/E05/RouterPingEachother.png)

---

## Lubuntu3  
![](/images/network-doc/E05/Lubuntu3Newaddress.png)  
![](/images/network-doc/E05/Lubuntu3Ipaddr.png)  
![](/images/network-doc/E05/Lubuntu3PhysicalMac.png)  
![](/images/network-doc/E05/Lubuntu3PingRouterB.png)

---

## Routing between routers

### Symmetric routing  
![](/images/network-doc/E05/StaticRouteRouterBLubuntu2.png)  
![](/images/network-doc/E05/StaticRouteRouterALubuntu3.png)  
![](/images/network-doc/E05/PingLubuntu2FromLubuntu3.png)  
![](/images/network-doc/E05/TracerouteLubuntu3toLubuntu2.png)

### Add full routing paths  
![](/images/network-doc/E05/RouterBtoLubuntu1AndSwitchA_B.png)

---

## Set up DHCP servers

### Router B (192.168.39.128/26)  
```bash
set service dhcp-server shared-network-name lubuntu3 subnet 192.168.39.128/26 default-router 192.168.39.129
set service dhcp-server shared-network-name lubuntu3 subnet 192.168.39.128/26 range PCs start 192.168.39.135
set service dhcp-server shared-network-name lubuntu3 subnet 192.168.39.128/26 range PCs stop 192.168.39.155
```

![](/images/network-doc/E05/RouterB_DHCP.png)  
![](/images/network-doc/E05/RouterBServiceconfig.png)

### Router A
```bash
# For 192.168.39.0/26
set service dhcp-server shared-network-name lubuntu1 subnet 192.168.39.0/26 default-router 192.168.39.1
set service dhcp-server shared-network-name lubuntu1 subnet 192.168.39.0/26 range PCs start 192.168.39.5
set service dhcp-server shared-network-name lubuntu1 subnet 192.168.39.0/26 range PCs stop 192.168.39.25

# For 192.168.39.64/26
set service dhcp-server shared-network-name lubuntu2 subnet 192.168.39.64/26 default-router 192.168.39.65
set service dhcp-server shared-network-name lubuntu2 subnet 192.168.39.64/26 range PCs start 192.168.39.70
set service dhcp-server shared-network-name lubuntu2 subnet 192.168.39.64/26 range PCs stop 192.168.39.90
```

![](/images/network-doc/E05/ShowCompareRouterA.png)  
![](/images/network-doc/E05/RouterAServiceconfig.png)

---

## Lubuntu IPv4 settings (DHCP)

### Lubuntu 1  
![](/images/network-doc/E05/Lubuntu1DHCPWorking.png)

### Lubuntu 2  
![](/images/network-doc/E05/Lubuntu2DHCPWorking.png)

### Lubuntu 3  
![](/images/network-doc/E05/Lubuntu3DHCPWorking.png)

### DHCP Leases  
![](/images/network-doc/E05/RouterALeasescircle.png)  
![](/images/network-doc/E05/RouterBLeases.png)

---

## Topology charts

### Physical  
![](/images/network-doc/E05/E05NetworkCharts-Physical_Topology.drawio.png)

### Logical  
![](/images/network-doc/E05/E05NetworkCharts-Logical_Topology.drawio.png)

---

## Connectivity tests

- Lubuntu3 to Lubuntu1  
  ![](/images/network-doc/E05/ConnectLubuntu3To1.png)

- Lubuntu2 to Lubuntu3  
  ![](/images/network-doc/E05/ConnectLubuntu2To3.png)

- Lubuntu1 to Lubuntu2  
  ![](/images/network-doc/E05/ConnectLubuntu1To2.png)

- Switch A to Lubuntu3  
  ![](/images/network-doc/E05/ConnectSwitchAtoLubuntu3.png)

- Switch B to Lubuntu2  
  ![](/images/network-doc/E05/ConnectSwitchAtoLubuntu2.png)

- Switch B to Lubuntu1  
  ![](/images/network-doc/E05/ConnectSwitchBToLubuntu1.png)

- Router B to Lubuntu1  
  ![](/images/network-doc/E05/ConnectRouterBLubuntu1.png)

- Router B to Switch B  
  ![](/images/network-doc/E05/ConnectRouterBSwitchB.png)

---

## Configuration files

- [Network_Switch_A](/images/network-doc/E05/Network_Switch_A.cfg)  
- [Network_Switch_B](/images/network-doc/E05/Network_Switch_b.cfg)  
- [Vyos Router A](/images/network-doc/E05/VyosRouterA.cfg)  
- [Vyos Router B](/images/network-doc/E05/VyosRouterB.cfg)
