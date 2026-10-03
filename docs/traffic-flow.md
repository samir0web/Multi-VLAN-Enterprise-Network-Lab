# Traffic Flow Example: PC9 to the WAN

1. PC9 (VLAN 32) sends the packet to its gateway `172.29.2.1`. SW1 tags the frame with VLAN 32 and sends it over the trunk.
2. R1 receives it on `Gi0/0/0.32` (NAT inside) and routes it toward `Gi0/0/1`.
3. ACL 100 matches, so PAT rewrites the source address to `20.20.20.1` with a unique port number.
4. ISP-L3 replies to `20.20.20.1`. R1 reverses the translation and forwards the reply to PC9.

## How the key features work

- **ACL 100** selects which traffic is translated. It excludes the server subnet, so the server never reaches the internet.
- **ACL 101** is applied inbound on `Gi0/0/1`. It drops packets from `20.20.20.0/24` headed to the server subnet. LAN traffic to the server never crosses that port, so it is unaffected.
- **PAT (`overload`)** lets many private hosts share one public IP, told apart by port numbers.
- **Native VLAN 200** carries untagged trunk frames in an unused VLAN, reducing VLAN-hopping risk. It must match on R1 (`native`) and SW1 (`native vlan 200`).
- **Exit-interface default route** works in Packet Tracer. In production, use a next hop: `ip route 0.0.0.0 0.0.0.0 20.20.20.2`.
