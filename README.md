# cisco-nat-vlan-ospf-labs
Cisco Packet Tracer labs covering enterprise network design — VLANs, inter-VLAN routing, OSPF, NAT/PAT, ACLs, and honeypot simulation. Built progressively from a small office network to a full NAT/security lab.

# Project 4: NAT + Internet Simulation with Honeypot 🔐

## Overview
A full enterprise network simulation built in Cisco Packet Tracer featuring internal department segmentation, a simulated ISP, and three types of NAT configuration. This lab also includes a honeypot server isolated from the internal network using ACLs to simulate a real-world security setup.

---

## Objectives
- Simulate a private corporate network connected to a public ISP
- Configure Static NAT, Dynamic NAT (concept), and PAT (NAT Overload)
- Isolate a honeypot server using Extended ACLs
- Understand how traffic flows across NAT boundaries

---

## Topology

```
[PC-Sales1/2]──[SW-Sales]──┐
[PC-HR1/2]────[SW-HR]──────┤
                            ├──[MLS-CORE (3560)]──[R-EDGE]──Serial──[R-ISP]──[Public-Server]
[Server-Internal]─[SW-Servers]─┤
[Server-Honeypot]─[SW-Honeypot]┘
```

---

## Network Addressing

| Segment | Subnet | VLAN | Gateway |
|---|---|---|---|
| Sales | 192.168.10.0/24 | 10 | 192.168.10.1 |
| HR | 192.168.20.0/24 | 20 | 192.168.20.1 |
| Servers | 192.168.30.0/24 | 30 | 192.168.30.1 |
| Honeypot | 192.168.40.0/24 | 40 | 192.168.40.1 |
| MLS-CORE ↔ R-EDGE | 192.168.100.0/30 | — | — |
| R-EDGE ↔ R-ISP | 200.0.0.0/30 | — | — |
| ISP/Public | 8.8.8.0/24 | — | — |

---

## NAT Configuration

### PAT (NAT Overload) — Sales & HR
All PCs in Sales (VLAN 10) and HR (VLAN 20) share a single public IP via port translation using R-EDGE's serial interface IP.

```
ip access-list standard PAT_TRAFFIC
 permit 192.168.10.0 0.0.0.255
 permit 192.168.20.0 0.0.0.255

ip nat inside source list PAT_TRAFFIC interface s0/3/0 overload
```

### Static NAT — Internal Server & Honeypot
One-to-one permanent mappings exposing internal servers to the public network.

```
ip nat inside source static 192.168.30.10 203.0.113.2
ip nat inside source static 192.168.40.10 203.0.113.3
```

---

## Honeypot Isolation (ACL)
The honeypot is reachable from the outside via Static NAT but is blocked from communicating with any internal department. ACL applied inbound on VLAN 40 SVI on MLS-CORE.

```
ip access-list extended HONEYPOT_ISOLATION
 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any

interface vlan 40
 ip access-group HONEYPOT_ISOLATION in
```

---

## Devices Used

| Device | Model | Role |
|---|---|---|
| MLS-CORE | Cisco 3560 | Layer 3 Switch / Inter-VLAN Routing |
| SW-Sales/HR/Servers/Honeypot | Cisco 2960 | Layer 2 Access Switches |
| R-EDGE | Cisco 2911 | NAT Gateway |
| R-ISP | Cisco 2911 | Simulated ISP Router |
| Public-Server | Server | Simulated Internet Server |

---

## Testing & Verification

| Test | Expected Result | Status |
|---|---|---|
| PC-Sales1 → 8.8.8.8 | ✅ Success (PAT) | Pass |
| PC-HR1 → 8.8.8.8 | ✅ Success (PAT) | Pass |
| Public-Server → 203.0.113.2 | ✅ Success (Static NAT) | Pass |
| Public-Server → 203.0.113.3 | ✅ Success (Static NAT) | Pass |
| Honeypot → PC-Sales1 | ❌ Blocked (ACL) | Pass |
| Honeypot → 8.8.8.8 | ✅ Success | Pass |
| Server-Internal → PC-Sales1 | ✅ Success | Pass |

---

## Key Learnings
- PAT allows thousands of internal hosts to share a single public IP using port numbers as identifiers
- Static NAT is ideal for servers that need to be consistently reachable from outside
- ACLs must be placed on the device closest to the traffic source — inter-VLAN traffic never reaches the edge router
- Wildcard masks in ACLs are the inverse of subnet masks and define which bits to ignore
- A default route (0.0.0.0/0) is essential on every device that needs to forward unknown traffic toward the internet

---

## Tools Used
- Cisco Packet Tracer
