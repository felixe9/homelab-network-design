# TP-Link ER605 — Network & Firewall Overview

> Last updated: 2026-05-29  
> Device: TP-Link ER605 · WAN1 uplink via FritzBox (192.168.178.9)

---

## Network zones

| VLAN | Name | Network | Gateway | Purpose |
|------|------|---------|---------|---------|
| 1 | Fritz / Legacy | 192.168.178.0/24 | 192.168.178.1 | ISP gateway · FritzBox uplink (WAN1) |
| 10 | Extended Network | 192.168.10.0/24 | 192.168.10.1 | Trusted clients · NAS |
| 20 | IoT Network | 192.168.20.0/24 | 192.168.20.1 | IoT devices |
| 31 | Camera System 1 | 192.168.31.0/24 | 192.168.31.1 | NVR 1 · Cam 11–12 |
| 32 | Camera System 2 | 192.168.32.0/24 | 192.168.32.1 | NVR 2 · Cam 21–24 |
| 90 | Guest Network | 192.168.90.0/24 | 192.168.90.1 | Guest Wi-Fi |
| 99 | Management | 192.168.99.0/24 | 192.168.99.1 | Infrastructure & admin access |

---

## Network isolation

### Isolated zones (outbound only)

Isolated zones **can**: reach the internet, use DNS & NTP.  
Isolated zones **cannot**: initiate connections to any other internal VLAN.  
Inbound access from Extended Network or Management remains possible via ACL rules.

| VLAN | Zone |
|------|------|
| 20 | IoT Network |
| 31 | Camera System 1 |
| 32 | Camera System 2 |
| 90 | Guest Network |

### Non-isolated zones

| VLAN | Zone | Reason |
|------|------|--------|
| 10 | Extended Network | Trusted internal network |
| 99 | Management | Admin access to all infrastructure required |

---

## IP groups

| Group name | Subnets / members | Notes |
|------------|-------------------|-------|
| Fritz_Net | 192.168.178.0/24 | FritzBox / WAN1 network |
| Extended_Net | 192.168.10.0/24 | Trusted clients, NAS |
| IoT_Net | 192.168.20.0/24 | IoT devices |
| CamSys1 | 192.168.31.0/24 | NVR 1, Cam 11–12 |
| CamSys2 | 192.168.32.0/24 | NVR 2, Cam 21–24 |
| Guest_Net | 192.168.90.0/24 | Guest Wi-Fi |
| Management | 192.168.99.0/24 | Infrastructure & admin |
| IPGROUP_LAN | 192.168.10.0/24, 192.168.20.0/24, 192.168.31.0/24, 192.168.32.0/24, 192.168.90.0/24, 192.168.99.0/24 | All internal VLANs (excl. Fritz) |

---

## ACL rules

Rules are processed in order. First match wins.

| # | Name | Policy | Direction | Service | Source | Destination | Purpose |
|---|------|--------|-----------|---------|--------|-------------|---------|
| 1 | Fritz_to_Router | Allow | WAN1 IN | ALL | Fritz_Net | Me (ER605) | FritzBox → ER605 management access |
| 2 | Fritz_to_Management | Allow | WAN1 IN | ALL ⚠️ | Fritz_Net | Management | FritzBox network → Management VLAN |
| 3 | Fritz_to_IoT | Allow | WAN1 IN | ALL | Fritz_Net | IoT_Net | FritzBox network → IoT VLAN 20 |
| 4 | Fritz_to_CamSys1 | Allow | WAN1 IN | ALL | Fritz_Net | CamSys1 | FritzBox network → Camera VLAN 31 |
| 5 | Fritz_to_CamSys2 | Allow | WAN1 IN | ALL | Fritz_Net | CamSys2 | FritzBox network → Camera VLAN 32 |
| 6 | Fritz_to_Extended | Allow | WAN1 IN | ALL | Fritz_Net | Extended_Net | FritzBox network → Extended Network VLAN 10 |
| 7 | Extended_to_CamSys1 | Allow | LAN → LAN | ALL | Extended_Net | CamSys1 | Trusted network → Camera VLAN 31 |
| 8 | Extended_to_CamSys2 | Allow | LAN → LAN | ALL | Extended_Net | CamSys2 | Trusted network → Camera VLAN 32 |
| ∞ | Default | **Deny** | — | ALL | ANY | ANY | Implicit deny — all unmatched traffic dropped |

> **⚠️ Rule 2 — Fritz_to_Management:**  
> The original config restricted this rule to HTTP only, which is insufficient for full management access.  
> Switches (192.168.99.10–12) and APs require SSH, HTTPS, SNMP, and ICMP in addition to HTTP.  
> This rule is currently set to **ALL**. Restrict to a defined service group if a tighter policy is required.

---

## Security model summary

| Zone | Internet | Can reach internal VLANs | Reachable from |
|------|----------|--------------------------|----------------|
| Extended Network (10) | ✓ | CamSys1 (31), CamSys2 (32) | Fritz_Net |
| IoT Network (20) | ✓ | None (isolated) | Fritz_Net |
| Camera System 1 (31) | ✓ | None (isolated) | Fritz_Net, Extended_Net, Management |
| Camera System 2 (32) | ✓ | None (isolated) | Fritz_Net, Extended_Net, Management |
| Guest Network (90) | ✓ | None (isolated) | — |
| Management (99) | ✓ | All (unisolated) | Fritz_Net |

---

## Static routing (FritzBox side)

Routes configured on the FritzBox to forward VLAN traffic to the ER605 (192.168.178.9):

```
Destination: 192.168.10.0   Mask: 255.255.255.0   Gateway: 192.168.178.9   # Extended Network
Destination: 192.168.20.0   Mask: 255.255.255.0   Gateway: 192.168.178.9   # IoT (optional)
Destination: 192.168.31.0   Mask: 255.255.255.0   Gateway: 192.168.178.9   # Camera System 1
Destination: 192.168.32.0   Mask: 255.255.255.0   Gateway: 192.168.178.9   # Camera System 2
Destination: 192.168.90.0   Mask: 255.255.255.0   Gateway: 192.168.178.9   # Guest (optional)
Destination: 192.168.99.0   Mask: 255.255.255.0   Gateway: 192.168.178.9   # Management (optional)
```

## Static routing (ER605 side)

```
Destination: 0.0.0.0   Mask: 0.0.0.0   Gateway: 192.168.178.1   # Default gateway
```
