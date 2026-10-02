# Multi-VLAN Enterprise Network Lab: Router-on-a-Stick, DHCP, NAT/PAT and WAN Link

A Cisco Packet Tracer lab that builds a small enterprise LAN behind one router, with inter-VLAN routing, DHCP for every VLAN, PAT with an extended ACL, a protected internal server, and a simulated ISP link.

![Packet Tracer](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-blue)
![Level](https://img.shields.io/badge/Level-CCNA-green)
![Topics](https://img.shields.io/badge/Topics-VLAN%20%7C%20RSTP%20%7C%20DHCP%20%7C%20NAT%20%7C%20ACL-orange)

---

## Table of Contents

1. [Overview](#1-overview)
2. [Topology](#2-topology)
3. [Design Notes and Corrections](#3-design-notes-and-corrections)
4. [Addressing Plan](#4-addressing-plan)
5. [Switch Port Map](#5-switch-port-map)
6. [Switch Configuration](#6-switch-configuration)
7. [Router (R1) Configuration](#7-router-r1-configuration)
8. [WAN Device Configuration](#8-wan-device-configuration)
9. [Traffic Flow Example](#9-traffic-flow-example)
10. [Verification](#10-verification)
11. [Troubleshooting Tips](#11-troubleshooting-tips)
12. [Repository Structure](#12-repository-structure)

---

## 1. Overview

One ISR4331 router (**R1**) and one Layer 2 switch (**SW1**) form the LAN. A 3650-24PS Layer 3 switch acts as the ISP router on the WAN side.

| Feature | Summary |
|---|---|
| Inter-VLAN routing | Router-on-a-stick, one trunk with sub-interfaces |
| DHCP | R1 serves all VLANs, including guest Wi-Fi |
| NAT | PAT (overload) using extended ACL 100, translated to `20.20.20.1` |
| Internal server | Own VLAN, reachable from the LAN only, blocked from the public network |
| Switching | RSTP, PortFast on edge ports, custom native VLAN, unused ports shut down |
| Routing | Static route to the WAN network plus a default route |

**Skills practiced:** VLANs, 802.1Q trunking, sub-interfaces, VLSM, DHCP, PAT, extended ACLs, static and default routing, RSTP, basic switch hardening.

---

## 2. Topology

![Topology](images/topology.png)

> Save your Packet Tracer screenshot as `images/topology.png`. The `.pkt` file goes in `/packet-tracer/`.

**Devices**

| Device | Model | Role |
|---|---|---|
| R1 | ISR4331 | Main router, DHCP, NAT, inter-VLAN routing |
| SW1 | Layer 2 switch (24 x FastEthernet) | Access and trunk switching |
| ISP-L3 | 3650-24PS | Simulated ISP/WAN router |
| PC1 to PC12 | PC-PT | End hosts |
| Server1 | Server-PT | Internal server |
| AP21 to AP24 | LAP-PT | Access points for guest Wi-Fi |

---

## 3. Design Notes and Corrections

The original task contained a few conflicts. This is how each was handled.

| # | Issue | Resolution |
|---|---|---|
| 1 | **VLAN 20 overlaps VLAN 30.** `172.29.2.208/28` (.208 to .223) sits inside `172.29.2.192/27` (.192 to .223). IOS rejects overlapping subnets on two interfaces. | VLAN 20 uses **172.29.2.224/28** (.224 to .239). |
| 2 | **VLAN 999 is outside the /22.** `172.29.99.0/30` is not within `172.29.0.0/22` (172.29.0.0 to 172.29.3.255). | Kept as given. It works because it is its own directly connected subnet. |
| 3 | The default route mentions **g1/0/1**, which is the ISP switch port. | R1 uses its own WAN port, **Gi0/0/1**. |
| 4 | The diagram shows APs inside staff VLAN circles, but the task assigns guest APs to VLAN 100. | AP ports (Fa0/21 to 24) are in **VLAN 100**. |

**Assumptions** (not visible in the diagram):

- SW1 trunk port: `Gi0/1`
- R1 LAN port: `Gi0/0/0`
- PCs and server are spread across `Fa0/1-13` as shown in the port map.

---

## 4. Addressing Plan

Network block: `172.29.0.0/22`

| VLAN | Purpose | Subnet | Gateway (R1) | Usable range |
|---|---|---|---|---|
| 32 | Staff (PC9 to PC12) | 172.29.2.0/25 | .1 | .2 to .126 |
| 31 | Staff (PC6 to PC8) | 172.29.2.128/26 | .129 | .130 to .190 |
| 30 | Staff (PC3 to PC5) | 172.29.2.192/27 | .193 | .194 to .222 |
| 20 | Staff (PC1 to PC2) | 172.29.2.224/28 | .225 | .226 to .238 |
| 100 | Guest Wi-Fi (APs) | 172.29.3.0/24 | .1 | .2 to .254 |
| 999 | Internal server | 172.29.99.0/30 | .1 | .2 (server) |
| 200 | Native VLAN | none | n/a | n/a |
| WAN | R1 to ISP-L3 | 20.20.20.0/24 | R1 `.1`, ISP-L3 `.2` | n/a |
| Loopback0 | R1 identity | 1.1.1.1/32 | n/a | n/a |

---

## 5. Switch Port Map

| Ports | Mode | VLAN | Notes |
|---|---|---|---|
| Gi0/1 | Trunk | Native 200 | Allowed: 20, 30, 31, 32, 100, 200, 999 (VLANs 1 and 50 blocked) |
| Fa0/1-2 | Access | 20 | PortFast |
| Fa0/3-5 | Access | 30 | PortFast |
| Fa0/6-8 | Access | 31 | PortFast |
| Fa0/9-12 | Access | 32 | PortFast |
| Fa0/13 | Access | 999 | Server, PortFast |
| Fa0/14-20 | Access | 50 | Unused, shut down |
| Fa0/21-24 | Access | 100 | Access points, PortFast |

---

## 6. Switch Configuration

<details>
<summary><b>SW1 full configuration (click to expand)</b></summary>

```
hostname SW1
!
! --- VLANs ---
vlan 20
 name STAFF-20
vlan 30
 name STAFF-30
vlan 31
 name STAFF-31
vlan 32
 name STAFF-32
vlan 50
 name UNUSED
vlan 100
 name GUEST-WIFI
vlan 200
 name NATIVE
vlan 999
 name INTERNAL-SERVER
!
! --- RSTP ---
spanning-tree mode rapid-pvst
!
! --- Trunk to router ---
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 200
 switchport trunk allowed vlan 20,30,31,32,100,200,999
 no shutdown
!
! --- End devices ---
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface range FastEthernet0/3-5
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
interface range FastEthernet0/6-8
 switchport mode access
 switchport access vlan 31
 spanning-tree portfast
interface range FastEthernet0/9-12
 switchport mode access
 switchport access vlan 32
 spanning-tree portfast
interface FastEthernet0/13
 switchport mode access
 switchport access vlan 999
 spanning-tree portfast
!
! --- Access points (guest) ---
interface range FastEthernet0/21-24
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
!
! --- Unused ports ---
interface range FastEthernet0/14-20
 switchport mode access
 switchport access vlan 50
 shutdown
```

</details>

**Why these settings**

| Setting | Reason |
|---|---|
| RSTP (`rapid-pvst`) | Converges in seconds instead of the 30 to 50 seconds of classic STP. |
| PortFast | Skips the listening and learning states on host and AP ports. Never enable it toward another switch. |
| Native VLAN 200 | Untagged trunk traffic goes to an unused VLAN instead of VLAN 1, which reduces VLAN-hopping risk. |
| `allowed vlan` list | This is what keeps VLANs 1 and 50 off the trunk. |
| Unused ports in VLAN 50 + `shutdown` | Anyone plugging in gets no network access. |

---

## 7. Router (R1) Configuration

### 7.1 Interfaces, sub-interfaces and loopback

```
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet0/0/0
 no shutdown
!
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 172.29.2.225 255.255.255.240
 ip nat inside
interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 172.29.2.193 255.255.255.224
 ip nat inside
interface GigabitEthernet0/0/0.31
 encapsulation dot1Q 31
 ip address 172.29.2.129 255.255.255.192
 ip nat inside
interface GigabitEthernet0/0/0.32
 encapsulation dot1Q 32
 ip address 172.29.2.1 255.255.255.128
 ip nat inside
interface GigabitEthernet0/0/0.100
 encapsulation dot1Q 100
 ip address 172.29.3.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/0/0.200
 encapsulation dot1Q 200 native
interface GigabitEthernet0/0/0.999
 encapsulation dot1Q 999
 ip address 172.29.99.1 255.255.255.252
 ip nat inside
!
! --- WAN ---
interface GigabitEthernet0/0/1
 ip address 20.20.20.1 255.255.255.0
 ip nat outside
 ip access-group 101 in
 no shutdown
```

> The `native` keyword on sub-interface `.200` must match the switch's native VLAN. A mismatch causes CDP/STP warnings and possible VLAN leaks.

### 7.2 DHCP

```
ip dhcp excluded-address 172.29.2.1
ip dhcp excluded-address 172.29.2.129
ip dhcp excluded-address 172.29.2.193
ip dhcp excluded-address 172.29.2.225
ip dhcp excluded-address 172.29.3.1
ip dhcp excluded-address 172.29.99.1
!
ip dhcp pool VLAN32
 network 172.29.2.0 255.255.255.128
 default-router 172.29.2.1
 dns-server 8.8.8.8
ip dhcp pool VLAN31
 network 172.29.2.128 255.255.255.192
 default-router 172.29.2.129
 dns-server 8.8.8.8
ip dhcp pool VLAN30
 network 172.29.2.192 255.255.255.224
 default-router 172.29.2.193
 dns-server 8.8.8.8
ip dhcp pool VLAN20
 network 172.29.2.224 255.255.255.240
 default-router 172.29.2.225
 dns-server 8.8.8.8
ip dhcp pool AP-GUEST-VLAN100
 network 172.29.3.0 255.255.255.0
 default-router 172.29.3.1
 dns-server 8.8.8.8
ip dhcp pool SERVER-VLAN999
 network 172.29.99.0 255.255.255.252
 default-router 172.29.99.1
```

Gateway addresses are excluded so DHCP never hands them to clients. The VLAN 999 pool has only one usable host address (`.2`), which goes to the server. You can also set it statically.

### 7.3 NAT with extended ACL 100

```
access-list 100 deny   ip 172.29.99.0 0.0.0.3 any
access-list 100 permit ip 172.29.0.0 0.0.3.255 any
!
ip nat inside source list 100 interface GigabitEthernet0/0/1 overload
```

- ACL 100 **selects which traffic gets translated**. It is not a firewall here.
- Line 1 excludes the server subnet, so it is never translated and has no internet access.
- Line 2 permits the whole LAN block (`172.29.0.0` to `172.29.3.255`).
- `overload` enables PAT: many private hosts share `20.20.20.1`, distinguished by port numbers.

### 7.4 Internal server protection

```
access-list 101 deny   ip 20.20.20.0 0.0.0.255 172.29.99.0 0.0.0.3
access-list 101 permit ip any any
```

Applied **inbound on Gi0/0/1** (see 7.1). It drops packets from the public `20.20.20.0/24` network headed to the server subnet. LAN users still reach the server because their traffic is routed internally and never crosses Gi0/0/1.

### 7.5 Routing

```
ip route 20.20.20.0 255.255.255.0 20.20.20.2
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0/1
```

- The static route sends the public network via the ISP-L3 switch.
- The default route sends all other unknown traffic out Gi0/0/1.
- On Ethernet, an exit interface makes the router ARP for every destination. It works in Packet Tracer, but in production use a next hop: `ip route 0.0.0.0 0.0.0.0 20.20.20.2`.

---

## 8. WAN Device Configuration

The 3650-24PS acts as the ISP router.

```
hostname ISP-L3
!
interface GigabitEthernet1/0/1
 no switchport
 ip address 20.20.20.2 255.255.255.0
 no shutdown
```

`no switchport` turns the port into a routed Layer 3 port so it can hold an IP address. Since `20.20.20.0/24` is directly connected on both ends, no extra routes are needed on this device.

---

## 9. Traffic Flow Example

**PC9 (VLAN 32) to the WAN**

1. PC9 sends the packet to its gateway `172.29.2.1`. SW1 tags it with VLAN 32 and sends it over the trunk.
2. R1 receives it on `Gi0/0/0.32` (NAT inside) and routes it toward `Gi0/0/1`.
3. ACL 100 matches, so PAT rewrites the source to `20.20.20.1` with a unique port.
4. ISP-L3 replies to `20.20.20.1`. R1 reverses the translation and forwards the reply to PC9.

---

## 10. Verification

| Test | Command / action | Expected result |
|---|---|---|
| VLAN membership | `show vlan brief` (SW1) | Ports in correct VLANs, Fa0/14-20 in VLAN 50 |
| Trunk | `show interfaces trunk` | Native 200, allowed list has no VLAN 1 or 50 |
| RSTP | `show spanning-tree summary` | Mode rapid-pvst |
| Sub-interfaces | `show ip interface brief` (R1) | All up/up |
| DHCP | `show ip dhcp binding` | Leases in every VLAN, including APs |
| Inter-VLAN | Ping PC3 to PC9 | Success |
| Server from LAN | Ping `172.29.99.2` from any PC | Success |
| Server from public | Ping `172.29.99.2` from ISP-L3 | Fails (ACL 101) |
| WAN | Ping `20.20.20.2` from any PC | Success |
| NAT | `show ip nat translations` | Inside global `20.20.20.1` |
| Loopback | `show ip route` | `1.1.1.1/32` connected |

---

## 11. Troubleshooting Tips

| Symptom | Likely cause | Check |
|---|---|---|
| PC gets `169.254.x.x` | DHCP not reaching the client | VLAN on access port, sub-interface up, pool network matches |
| No inter-VLAN routing | Trunk or sub-interface problem | `show interfaces trunk`, `encapsulation dot1Q` IDs match VLANs |
| Native VLAN mismatch warnings | Different native VLAN on each side | `native 200` on R1 and `native vlan 200` on SW1 |
| No internet from LAN | NAT roles or ACL wrong | `ip nat inside/outside` on interfaces, ACL 100 order |
| Server reachable from WAN | ACL 101 missing or misapplied | `show ip interface Gi0/0/1` for inbound ACL |
| Port stuck in err-disabled | Unused port used or shut down | `show interfaces status` |

---

## 12. Repository Structure

```
.
├── README.md
├── images/
│   └── topology.png
├── packet-tracer/
│   └── multi-vlan-nat-lab.pkt
└── configs/
    ├── SW1.txt
    ├── R1.txt
    └── ISP-L3.txt
```

---

## Author

**Samir** · Networking student, CCNA
Feel free to open an issue or pull request if you spot an error or have an improvement.
