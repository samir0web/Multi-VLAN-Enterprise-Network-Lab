# Troubleshooting

| Symptom | Likely cause | Check |
|---|---|---|
| PC gets `169.254.x.x` | DHCP not reaching the client | VLAN on access port, sub-interface up, pool network matches |
| No inter-VLAN routing | Trunk or sub-interface problem | `show interfaces trunk`, `encapsulation dot1Q` IDs match VLANs |
| Native VLAN mismatch warnings | Different native VLAN on each side | `native 200` on R1 and `native vlan 200` on SW1 |
| No internet from LAN | NAT roles or ACL wrong | `ip nat inside/outside` on interfaces, ACL 100 order |
| Server reachable from WAN | ACL 101 missing or misapplied | `show ip interface Gi0/0/1` for inbound ACL |
| Port in err-disabled or down | Unused port or shut down | `show interfaces status` |
