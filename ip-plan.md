# IP adress overview

## General overview of IP address ranges

  | Zone | Network | Range | Device |
  |---|---|---|---|
  | FritzBox (Legacy Zone / Flat Zone) | 192.168.178.0/24 | 192.168.178.1 – 192.168.178.254 | FritzBox = .1, ER605 = .9 |
  | Management | 192.168.99.0/24 | — | VLAN 99, same range |
  | Extended network | 192.168.10.0/24 | 192.168.10.1 – .254 | NAS = .10 |
  | IoT | 192.168.20.0/24 | 192.168.20.1 – .254 | AP = .10 |
  | Cameras Sys1 | 192.168.31.0/24 | 192.168.31.1 – .254 | Cam11 = .11, Cam12 = .12 |
  | Cameras Sys2 | 192.168.32.0/24 | 192.168.32.1 – .254 | Cam11-14 = .11–.14 |
  | Guest | 192.168.90.0/24 | 192.168.90.1 – .254 | — |



## Overview of the device network

### FritzBox(ISP Gateway)

#### IP configuration:
| **Device** | Networkadress | Subnet | Interface | Configuration |
|---|---|---|---|---|
|**Fritzbox Router**|192.168.178.1|24|LAN / WLAN|DHCP|
||||
|**Static addresses:**|192.168.178.1-15||all local|
|**Fritz!Mesh Repeater1**|192.168.178.2|/24| wlan0 (2.4 GHz) / wlan1 (5 GHz)|Mesh|
|**Fritz!Mesh Repeater2**|192.168.178.3|/24| wlan0 (2.4 GHz) / wlan1 (5 GHz)|Mesh|
|**TP-Link (Router)**|192.168.178.9|/24| lan0 |static routes| 

#### Static routing configuration:
```
Destination: 192.168.10.0   Mask: 255.255.255.0   Gateway: 192.168.178.9  Extended network
Destination: 192.168.20.0   Mask: 255.255.255.0   Gateway: 192.168.178.9  Iot //optional
Destination: 192.168.31.0   Mask: 255.255.255.0   Gateway: 192.168.178.9  Surveillance 1
Destination: 192.168.32.0   Mask: 255.255.255.0   Gateway: 192.168.178.9  Surveillance 2
Destination: 192.168.90.0   Mask: 255.255.255.0   Gateway: 192.168.178.9  guest //optional
Destination: 192.168.99.0   Mask: 255.255.255.0   Gateway: 192.168.178.9  //optional
```

---
### TP-Link (Router)

#### Uplink
  | Parameter              | Wert                          |
  |------------------------|-------------------------------|
  | WAN/LAN (Uplink)       | 192.168.178.9                 |
  | Gateway                | 192.168.178.1                 |
  | Rolle                  | VLAN Routing / Core Router    |
  | NAT                    | enable (double Nat)           |

#### VLAN Interfaces
  | VLAN | Interface IP       | Beschreibung              |
  |------|--------------------|---------------------------|
  | 99   | 192.168.99.1       | Management                |
  | 10   | 192.168.10.1       | NAS                       |
  | 20   | 192.168.20.1       | IoT                       |
  | 31   | 192.168.31.1       | Recorder 1                |
  | 32   | 192.168.32.1       | Recorder 2                |
  | 90   | 192.168.90.1       | Guest                     |

#### Network and VLAN Mapping
  | VLAN | Interface | Name                | Netzwerk           | Gateway        | DHCP        | Zweck                          |
  |------|-----------|---------------------|-------------------|----------------|--------------|--------------------------------|
  | 1    | 01        | Default / Fritz     | 192.168.178.0/24  | 192.168.178.1  | FritzBox     | Internet / Basisnetz           |
  | 10   | 02,       | Extended network    | 192.168.10.0/24   | 192.168.10.1   | not provided | Extended network               |
  | 20   | 02,03     | IoT                 | 192.168.20.0/24   | 192.168.20.1   | ER605        | IoT / Testgeräte               |
  | 31   | 02,       | Surveillance 1      | 192.168.31.0/24   | 192.168.31.1   | not provided | Surveillance 1                 |
  | 32   | 02,03     | Surveillance 2      | 192.168.32.0/24   | 192.168.32.1   | not provided | Surveillance 2                 |
  | 90   | 02,03     | Guest               | 192.168.90.0/24   | 192.168.90.1   | ER605        | Gäste WLAN                     |
  | 99   | 02,03,05  | Management          | 192.168.99.0/24   | 192.168.99.1   | ER605        | Infrastruktur / Admin Zugriff  |

#### Static routing configuration:
```
Destination: 0.0.0.0   Mask: 0.0.0.0   Gateway: 192.168.178.1  default gateway
```
---
### Switch 1
IP: 192.168.99.10/24
| Port | Typ      | VLANs                     | Beschreibung                  |
|------|----------|---------------------------|-------------------------------|
| 1    | Trunk    | 10,20,31,32,90,99   (Tag) | Uplink to ER605               |
| 2    | Access   | VLAN 20                   | IoT AP                        |
| 3    | Access   | VLAN 10                   | Extended network              |
| 4    | Access   | VLAN 20                   | IoT (reserve)                 |
| 5    | Access   | VLAN 20                   | IoT (reserve)                 |
| 6    | Trunk    | VLAN 31,99                | Uplink to Cam 11-12           |
| 7    | Access   | VLAN 31                   | Surveillance 1 / NVR 1        |
| 8    | Access   | VLAN 32                   | Surveillance 2 / NVR 2        |

---
### Switch 2
IP: 192.168.99.11/24

| Port | Typ      | VLANs                     | Beschreibung                  |
|------|----------|---------------------------|-------------------------------|
| 1    | Trunk    | 31,99                     | Uplink                        |
| 2    | Access   | VLAN 31                   |     Cam 11 ->(NVR 1)          |
| 3    | Access   | VLAN 31                   |     Cam 12 ->(NVR 1)          |

---

### Switch 3
IP: 192.168.99.12/24
| Port | Typ      | VLANs                     | Beschreibung                  |
|------|----------|---------------------------|-------------------------------|
| 1    | Access  | VLAN 32                   | NVR 2 / Cam 21                 |
| 2    | Access  | VLAN 32                   | NVR 2 / Cam 22                 |
| 3    | Access  | VLAN 32                   | NVR 2 / Cam 23                 |
| 4    | Access  | VLAN 32                   | NVR 2 / Cam 24                 |
| 5    | Access  | VLAN 1(default)           |
| 6    | Access  | VLAN 20,                  | Solar Logger                   |
| 7    | Trunk   | VLAN 20,32,90,99          | Uplink zum ER605               |
| 8    | Access  | VLAN 1(default)           |
  

### NAS
| Gerät | IP                    | VLAN | Gateway       | Beschreibung |
|------|------------------------|------| ------------- |--------------|
| NAS  | 192.168.10.10          | 10   | 192.168.10.1  |              |

### IoT
| Gerät     | IP                | VLAN |Gateway       | Beschreibung |
|-----------|-------------------|------| ------------ |-------------|
| IoT AP    | 192.168.20.10     | 20   | 192.168.20.1 | Access Point |

### Surveillance
#### Cam's isolated in VLAN 31/32
| Gerät           | IP               | VLAN | Gateway        |Beschreibung     | 
|-----------------|------------------|------|----------------|-----------------|
|     NVR 1       | 192.168.31.10    | 31   | 192.168.31.1   | lan0/0          |
|     NVR 2       | 192.168.32.10    | 32   | 192.168.32.1   | lan1/0          |
||||||
| Cam 11          | 192.168.31.11    | 31   | 192.168.31.1  | Cam 11           |
| Cam 12          | 192.168.31.12    | 31   | 192.168.31.1  | Cam 12           |
||||||
| Cam 21          | 192.168.32.11    | 32   | 192.168.32.1  | Cam 21           |
| Cam 22          | 192.168.32.12    | 32   | 192.168.32.1  | Cam 22           |
| Cam 23          | 192.168.32.13    | 32   | 192.168.32.1  | Cam 23           |
| Cam 24          | 192.168.32.14    | 32   | 192.168.32.1  | Cam 24           |




