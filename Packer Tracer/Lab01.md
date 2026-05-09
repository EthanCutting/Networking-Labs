# Enterprise Branch Network Lab

## Lab Topology
<img width="1144" height="845" alt="Screenshot 2026-05-09 103842" src="https://github.com/user-attachments/assets/1063de00-5448-49b6-a5ab-f9f56e5fbf88" />

## Overview 

This project is a Cisco Packet Tracer enterprise branch network lab designed to practise core networking concepts used in real business environments.

The lab includes VLANs, VTP, trunking, EtherChannel, inter-VLAN routing, OSPF, default routing, NAT/PAT, ACLs, and server connectivity. The goal was to build a multi-VLAN branch network where end devices can communicate through a Layer 3 core switch, reach an edge router, and access an external simulated internet server.

### Main Devices 

| Device | Role |
|---|---|
| CORE-SW1 | Layer 3 core switch, VLAN gateways, inter-VLAN routing, OSPF |
| EDGE-R1 | Edge router, NAT/PAT, default route to ISP |
| ISP-R1 | Simulated ISP router |
| INET-SRV | Simulated internet/DNS server |
| BR1-SW1 | Branch access switch |
| BR2-SW1 | Branch access switch |
| SERVER-SW1 | Server access switch |
| LAN-SRV1 | Internal LAN server |
| BR1-PC1 | Branch 1 client PC |
| BR2-PC2 | Branch 2 client PC |

---

## Features Configured

- VTP domain for VLAN management
- Multiple VLANs across the network
- 802.1Q trunk links between switches
- Layer 2 EtherChannel using LACP
- Inter-VLAN routing on `CORE-SW1`
- Routed link between `CORE-SW1` and `EDGE-R1`
- OSPF routing between the core switch and edge router
- Default route from the core toward the edge router
- NAT/PAT overload on the edge router
- ACL to restrict traffic between VLANs
- Static IP addressing on servers
- End-to-end connectivity testing

---

## VLAN Design

| VLAN | Name / Purpose | Subnet | Default Gateway |
|---|---|---|---|
| VLAN 10 | Users / Branch 1 | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | Admin | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | HR / Branch 2 | 192.168.30.0/24 | 192.168.30.1 |
| VLAN 40 | Accounting | 192.168.40.0/24 | 192.168.40.1 |
| VLAN 50 | Servers | 192.168.50.0/24 | 192.168.50.1 |
| VLAN 99 | Management | 192.168.99.0/24 | 192.168.99.1 |

---

## IP Addressing Summary

| Link / Device | IP Address |
|---|---|
| CORE-SW1 to EDGE-R1 | 10.0.0.0/30 |
| CORE-SW1 routed port | 10.0.0.2/30 |
| EDGE-R1 inside interface | 10.0.0.1/30 |
| EDGE-R1 outside interface | 203.0.113.2/30 |
| ISP-R1 interface to EDGE-R1 | 203.0.113.1/30 |
| ISP-R1 interface to INET-SRV | 8.8.8.1/16 |
| INET-SRV | 8.8.8.8/16 |
| INET-SRV gateway | 8.8.8.1 |

---


