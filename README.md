# OSPF + Inter-VLAN Routing Lab (Cisco Packet Tracer)

A Cisco Packet Tracer lab demonstrating inter-VLAN routing via Layer 3 Switch SVIs, OSPF Area 0 across three devices, and DHCP relay across VLAN boundaries.

---

## Topology

<img width="1658" height="587" alt="image" src="https://github.com/user-attachments/assets/e78539f0-4ec9-403f-a334-45e554a3f5bd" />

---

## Addressing Table

| Device | Interface | IP Address | Subnet | Notes |
|---|---|---|---|---|
| R1 | Gi6/0 | 10.0.12.1 | /30 | Link to SW1 |
| SW1 | Gi1/1/1 | 10.0.12.2 | /30 | Link to R1 |
| SW1 | Gi1/1/2 | 10.0.23.2 | /30 | Link to R2 |
| SW1 | Vlan10 SVI | 192.168.10.1 | /24 | Users gateway |
| SW1 | Vlan20 SVI | 192.168.20.1 | /24 | Servers gateway |
| SW1 | Vlan30 SVI | 192.168.30.1 | /24 | Mgmt gateway |
| R2 | Gi6/0 | 10.0.23.1 | /30 | Link to SW1 |
| DHCP Server | NIC | 192.168.20.2 | /24 | Static, VLAN 20 |
| WEB Server | NIC | 192.168.20.3 | /24 | Static, VLAN 20 |
| PC6–PC12 | NIC | DHCP | /24 | VLAN 10 |
| PC8–PC9 | NIC | DHCP | /24 | VLAN 30 |

---

## VLAN Assignment

| VLAN | Name | Subnet | Access Ports (SW1) | Hosts |
|---|---|---|---|---|
| 10 | Users | 192.168.10.0/24 | Gi1/0/1, Gi1/0/2, Gi1/0/5 | PC6, PC7, PC12 |
| 20 | Servers | 192.168.20.0/24 | Gi1/0/6, Gi1/0/7 | DHCP Server, WEB Server |
| 30 | Mgmt | 192.168.30.0/24 | Gi1/0/3, Gi1/0/4 | PC8, PC9 |

---

## Features Demonstrated

- **Inter-VLAN routing via SVIs** — SW1 acts as the Layer 3 gateway for all three VLANs without a separate router-on-a-stick setup
- **OSPF Area 0** — single-area OSPF running across R1, SW1, and R2, with SW1 advertising all VLAN subnets and both point-to-point links
- **DHCP relay (ip helper-address)** — VLAN 10 and VLAN 30 hosts receive dynamic IPs from a centralised DHCP server in VLAN 20, relayed by SW1's SVIs
- **Passive interfaces** — VLAN SVIs marked passive in OSPF so hello packets are not sent toward end hosts
- **PortFast** — enabled on all access ports to skip STP listening/learning for faster host connectivity
- **Full end-to-end reachability** — any host in any VLAN can reach any other host, both routers, and both servers

---

## Key Concepts

### Why a Multilayer Switch for Inter-VLAN Routing?

A Layer 3 switch with SVIs is far more efficient than router-on-a-stick. Each VLAN gets a virtual interface directly on the switch — no trunk link back to a router required, and traffic between VLANs is switched in hardware at wire speed.

### Why OSPF?

OSPF lets R1 and R2 dynamically learn the three VLAN subnets from SW1 without static routes. If a subnet changes or a link goes down, OSPF reconverges automatically. This simulates a real enterprise edge where branch routers need to know about internal segments.

### Why DHCP Relay?

Rather than running a separate DHCP server per VLAN, a single server in VLAN 20 serves all VLANs. SW1's `ip helper-address` on each SVI forwards broadcast DHCP Discover packets as unicast to `192.168.20.2`, simulating a centralised IP management model used in real networks.

---

## OSPF Configuration Summary

| Device | Router ID | Networks Advertised |
|---|---|---|
| R1 | 1.1.1.1 | 10.0.12.0/30 |
| SW1 | 2.2.2.2 | 10.0.12.0/30, 10.0.23.0/30, 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24 |
| R2 | 3.3.3.3 | 10.0.23.0/30 |

---

## Verification Commands

```bash
# On SW1 — confirm OSPF adjacencies
show ip ospf neighbor
# Expected: R1 (1.1.1.1) FULL, R2 (3.3.3.3) FULL

# On R1 or R2 — confirm VLAN routes learned via OSPF
show ip route ospf
# Expected: O 192.168.10.0/24, O 192.168.20.0/24, O 192.168.30.0/24

# On SW1 — confirm all interfaces up
show ip interface brief

# On SW1 — confirm VLAN 
show vlan brief

# From PC6 (VLAN 10) — cross-VLAN ping
ping 192.168.20.2     # DHCP server
ping 192.168.30.10    # PC8 in Mgmt VLAN
ping 10.0.12.1        # R1
ping 10.0.23.1        # R2 (full path through OSPF)

# From R1 — traceroute to a VLAN 30 host
traceroute 192.168.30.10
```

---

## Files

```
ospf-intervlan-lab.pkt  
README.md
```

---

## Requirements

- Cisco Packet Tracer
- No physical hardware needed

---

## Skills Practised

`Cisco IOS` · `VLANs` · `SVI` · `Inter-VLAN Routing` · `OSPF` · `DHCP Relay` · `ip helper-address` · `Layer 3 Switching` · `Spanning Tree PortFast` · `Network Troubleshooting`
