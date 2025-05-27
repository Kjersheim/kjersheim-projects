---
title: "Data Networks Episode 6: Loop Detection – Ethernet and Spanning Tree"
date: 2023-02-17
draft: false
summary: "Exploring Ethernet switching behavior, loops in RING topologies, and enabling Spanning Tree Protocol to mitigate broadcast storms."
---

## Plan

- [x] Update and review current physical topology  
- [x] Devices (Workstations, Network devices, Racks, Cabling)  
- [x] Rack Diagram  
- [x] Budget Table  
- [ ] Device Rack Documentation (next step)

---

## Updated Physical Topology

### Conceptual office layout  
![](/images/network-doc/E06/PhysicalOfficeDraft1.png)

### Proper physical topology (based on E05)  
![](/images/network-doc/E06/ProperPhysicalTopology.png)

---

## Devices

### Workstations

- Ultrabook laptops (Lenovo X1 Nano) with docking stations (Ugreen)
- Peripherals: screens, keyboards, mice

![](/images/network-doc/E06/UgreenDockingStation.png)

---

### Switches

- Cisco Catalyst 1000-16T (x2)
- Mounting kits and rack compatibility considerations

![](/images/network-doc/E06/Cisco1.png)
![](/images/network-doc/E06/Cisco2.png)
![](/images/network-doc/E06/Cisco3.png)
![](/images/network-doc/E06/Rackmount.png)
![](/images/network-doc/E06/Rackmount2.png)
![](/images/network-doc/E06/Rackmount3.png)

---

### Routers

After much research and consideration:
- Cisco ISR1100 chosen over home-grade or discontinued options
- Planned fiber module included for ISP connectivity

![](/images/network-doc/E06/Router3.png)
![](/images/network-doc/E06/Fibermodule.png)
![](/images/network-doc/E06/RouterOptions.png)
![](/images/network-doc/E06/EndRouter.png)

---

### Modem

No separate modem selected—ISP fiber assumed terminated directly into router via SFP module.

---

## Device Cabinets

- StarTech 12U sideways wall rack  
![](/images/network-doc/E06/MediumRack.png)

- Shelf units  
![](/images/network-doc/E06/1UShelf.png)
![](/images/network-doc/E06/1UShelf2.png)

- RJ45 Patch Panels (x2)  
![](/images/network-doc/E06/RJ45Panel.png)

- Fiber patch panel with mounting kit  
![](/images/network-doc/E06/RackPower.png)

---

## Cabling

### From data-room to departments  
- 300m Cat6a shielded: ~330€

![](/images/network-doc/E06/HouseCables1.png)

### In-rack cabling  
- 1x 2m fiber LC cable  
- 3x 1.5m Cat6a cables  
- 3x 1m Cat6a cables (router-router, switch-switch, etc.)

---

## Rack Diagram

![](/images/network-doc/E06/RackDrawing1.png)

---

## Network Budget

### Workstations

| Item | Qty | € each | Total |
|------|-----|--------|-------|
| Docking station (Ugreen) | 3 | 309.50 | 928.50 |
| Lenovo X1 Nano | 3 | 2000 | 6000 |
| Accessories (monitor, mouse, kb) | 3 | 1000 | 3000 |
| **Total** |  |  | **9,928.50** |

### Network Devices

| Item | Qty | € each | Total |
|------|-----|--------|-------|
| Cisco Catalyst 1000-16T | 2 | 616.24 | 1,232.48 |
| Switch rack kits | 2 | 150 | 300 |
| Cisco ISR router | 2 | 511.16 | 1,022.32 |
| 12U Rack (StarTech) | 1 | 507.40 | 507.40 |
| RJ45 patch panel | 2 | 30 | 60 |
| Fiber patch panel | 1 | 100 | 100 |
| Mounting kit (fiber) | 1 | 30 | 30 |
| Rack power strip | 1 | 30 | 30 |
| **Total** |  |  | **3,282.20** |

### Grand Total

| Category | € |
|----------|----|
| Workstations | 9,928.50 |
| Network setup | 3,282.20 |
| **Total** | **13,210.70** |
