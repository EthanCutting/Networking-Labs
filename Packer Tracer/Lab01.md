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

## Device Configuration Summary 
This section explains what I configured on each switch and router in the lab, and what role each device has in the network.

---

# Switch Configuration 

## CORE-SW1 Configuration 
`CORE-SW1` is the main Layer 3 switch in this lab. I used it as the central switch for VLAN routing, trunking, EtherChannel, OSPF, and forwarding traffic toward the edge router.

### Commands Configured on CORE-SW1  
```cisco
ip routing
```
I enabled ip routing so the multilayer switch could route traffic between VLANs. This allows devices in different VLANs, such as VLAN 10 and VLAN 30, to communicate through their default gateways on CORE-SW1.

```cisco
spanning-tree mode pvst
```
I used PVST spanning tree mode to help prevent Layer 2 loops in the switched network. This is useful when multiple switches and trunk links are used.

```cisco
interface FastEthernet0/3
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface FastEthernet0/4
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
```
I configured Fa0/3 and Fa0/4 as trunk ports. These links carry multiple VLANs between CORE-SW1 and the branch/access switches. I also limited the trunk to only allow VLANs 10, 20, 30, 40, 50, and 99.

```cisco
interface FastEthernet0/5
 no switchport
 ip address 10.0.0.2 255.255.255.252
 duplex auto
 speed auto
```
I configured Fa0/5 as a routed port instead of a normal switchport. This port connects CORE-SW1 to EDGE-R1 using the point-to-point network 10.0.0.0/30.

```cisco
interface FastEthernet0/23
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active

interface FastEthernet0/24
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
```
I configured Fa0/23 and Fa0/24 as trunk ports and added them to EtherChannel group 1 using LACP active mode. This combines two physical links into one logical link for better redundancy and bandwidth.

```cisco
interface Port-channel1
 switchport trunk allowed vlan 10,20,30,40,50,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
```
I configured Port-channel1 as a trunk. This is the logical EtherChannel interface created from Fa0/23 and Fa0/24. VLAN traffic can pass through this port-channel to the connected access/server switch.


```cisco
interface Vlan10
 mac-address 0040.0bd9.bc01
 ip address 192.168.10.1 255.255.255.0

interface Vlan20
 mac-address 0040.0bd9.bc02
 ip address 192.168.20.1 255.255.255.0

interface Vlan30
 mac-address 0040.0bd9.bc03
 ip address 192.168.30.1 255.255.255.0

interface Vlan40
 mac-address 0040.0bd9.bc04
 ip address 192.168.40.1 255.255.255.0

interface Vlan50
 mac-address 0040.0bd9.bc05
 ip address 192.168.50.1 255.255.255.0

interface Vlan99
 mac-address 0040.0bd9.bc06
 ip address 192.168.99.1 255.255.255.0
```
I created VLAN interfaces, also called SVIs, for each VLAN. These act as the default gateways for devices inside each VLAN.

| VLAN    | Gateway IP   |
| ------- | ------------ |
| VLAN 10 | 192.168.10.1 |
| VLAN 20 | 192.168.20.1 |
| VLAN 30 | 192.168.30.1 |
| VLAN 40 | 192.168.40.1 |
| VLAN 50 | 192.168.50.1 |
| VLAN 99 | 192.168.99.1 |

This allows CORE-SW1 to perform inter-VLAN routing.

```cisco
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 192.168.50.0 0.0.0.255 area 0
 network 192.168.99.0 0.0.0.255 area 0
 default-information originate
```
I configured OSPF on CORE-SW1 using process ID 1 and router ID 1.1.1.1. I advertised the routed link to EDGE-R1 and all internal VLAN networks into OSPF.

The default-information originate command allows CORE-SW1 to advertise a default route into OSPF, so other OSPF routers can learn where to send unknown traffic.

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```
I configured a default route pointing to EDGE-R1 at 10.0.0.1. This means any traffic that does not match a local route will be sent toward the edge router.

```cisco
ip access-list extended BLOCK_HR_TO_USERS
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any
```
I created an extended ACL called BLOCK_HR_TO_USERS. The purpose of this ACL is to block traffic from the HR VLAN 192.168.30.0/24 going to the Users VLAN 192.168.10.0/24.

The final permit ip any any line allows all other traffic so only HR-to-Users traffic is blocked.

Note: The ACL has been created, but to fully enforce it, it should be applied to an interface using an ip access-group command.

Example:
```cisco
interface Vlan30
 ip access-group BLOCK_HR_TO_USERS in
```

Summary of What I Did on CORE-SW1

On CORE-SW1, I configured the main Layer 3 switching functions for the network. I enabled IP routing so the switch could route between VLANs, then created VLAN interfaces for VLANs 10, 20, 30, 40, 50, and 99. Each VLAN interface acts as the default gateway for devices in that VLAN.

I configured trunk ports to carry multiple VLANs between the core switch and the access switches. I also configured EtherChannel using Fa0/23 and Fa0/24, which combines two physical links into one logical trunk link.

I configured Fa0/5 as a routed port to connect the core switch to EDGE-R1. This allows the internal network to forward traffic toward the edge router and eventually out to the simulated internet.

I also configured OSPF so CORE-SW1 can advertise all internal VLAN networks and the routed link to the edge router. A default route was added to send unknown traffic toward EDGE-R1.

Finally, I created an ACL to block traffic from the HR VLAN to the Users VLAN. This demonstrates basic internal network segmentation and access control.
