# Addressing Plan

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

## Switch port map

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

## Corrections to the original task

| # | Issue | Resolution |
|---|---|---|
| 1 | VLAN 20 `172.29.2.208/28` (.208 to .223) sits inside VLAN 30 `172.29.2.192/27` (.192 to .223). IOS rejects overlapping subnets. | VLAN 20 uses `172.29.2.224/28` (.224 to .239). |
| 2 | VLAN 999 `172.29.99.0/30` is outside `172.29.0.0/22` (172.29.0.0 to 172.29.3.255). | Kept as given. It works as its own directly connected subnet. |
| 3 | The default route mentions `g1/0/1`, which is the ISP switch port. | R1 uses its own WAN port `Gi0/0/1`. |
| 4 | The diagram shows APs inside staff VLAN circles, but the task assigns guest APs to VLAN 100. | AP ports (Fa0/21 to 24) are in VLAN 100. |

## Assumptions

These labels are not visible in the diagram:

- SW1 trunk port: `Gi0/1`
- R1 LAN port: `Gi0/0/0`
- PCs and server are spread across `Fa0/1-13` as in the port map above.
