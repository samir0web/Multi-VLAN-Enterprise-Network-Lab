# Verification Tests

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
