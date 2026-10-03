# Original Requirements

**Network ID:** 172.29.0.0/22

| VLAN | Subnet |
|---|---|
| 32 | 172.29.2.0/25 |
| 31 | 172.29.2.128/26 |
| 30 | 172.29.2.192/27 |
| 20 | 172.29.2.208/28 (corrected to .224/28, see [addressing plan](../topology/addressing-plan.md)) |
| 999 | 172.29.99.0/30 (Internal Server) |
| 100 | 172.29.3.0/24 (Access Points for guest users) |

## Router

- DHCP server for all VLANs, plus access point DHCP
- NAT with extended ACL (access-list 100)
- Loopback address: 1.1.1.1/32
- Internal server access: no access from the public network (20.20.20.0/24), LAN only
- Static route to 20.20.20.0 with gateway 20.20.20.2
- Default route 0.0.0.0 0.0.0.0 via exit interface

## Switch

- Native VLAN 200
- RSTP
- PortFast for end devices and access points
- Trunk port
- Access ports for PCs and server (Fa0/1-13) and access points (Fa0/21-24)
- Unused ports (Fa0/14-20) in VLAN 50 and in shutdown state
- VLANs 1 and 50 blocked from the trunk

## WAN

- Add a router (an L3 switch in this lab) and connect it to the main router
- Public IP 20.20.20.1/24 on R1 (Gi0/0/1) and 20.20.20.2/24 on the L3 switch (Gi1/0/1)
