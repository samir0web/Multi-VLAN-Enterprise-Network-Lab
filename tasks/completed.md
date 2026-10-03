# Task Completion Checklist

## Router

| Task | Status | Where |
|---|---|---|
| DHCP for all VLANs and APs | Done | [R1.txt](../configs/R1.txt) |
| NAT with extended ACL 100 | Done | [R1.txt](../configs/R1.txt) |
| Loopback 1.1.1.1/32 | Done | [R1.txt](../configs/R1.txt) |
| Server blocked from public network | Done | [R1.txt](../configs/R1.txt) (ACL 101) |
| Static route to 20.20.20.0 via 20.20.20.2 | Done | [R1.txt](../configs/R1.txt) |
| Default route via Gi0/0/1 | Done | [R1.txt](../configs/R1.txt) |

## Switch

| Task | Status | Where |
|---|---|---|
| Native VLAN 200 | Done | [SW1.txt](../configs/SW1.txt) |
| RSTP | Done | [SW1.txt](../configs/SW1.txt) |
| PortFast on end devices and APs | Done | [SW1.txt](../configs/SW1.txt) |
| Trunk port | Done | [SW1.txt](../configs/SW1.txt) |
| Access ports (Fa0/1-13, Fa0/21-24) | Done | [SW1.txt](../configs/SW1.txt) |
| Unused ports in VLAN 50, shut down | Done | [SW1.txt](../configs/SW1.txt) |
| VLANs 1 and 50 blocked from trunk | Done | [SW1.txt](../configs/SW1.txt) |

## WAN

| Task | Status | Where |
|---|---|---|
| R1 Gi0/0/1 = 20.20.20.1/24 | Done | [R1.txt](../configs/R1.txt) |
| L3 switch Gi1/0/1 = 20.20.20.2/24 | Done | [ISP-L3.txt](../configs/ISP-L3.txt) |
