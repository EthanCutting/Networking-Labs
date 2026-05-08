# GNS3 LAB 02 - Enterprise Network Setup

This lab demonstrates a small enterprise network in GNS3 using two switches, VLAN segmentation, inter-VLAN routing, DHCP, and NAT.

---
## Network Topology
<img width="934" height="491" alt="lab02" src="https://github.com/user-attachments/assets/1c721b5a-6786-41f8-9d44-b0cf681c103a" />

PCs / Ubuntu -> SW2 -> SW1 -> R1 -> R2 -> Internet

## Subnet Plan

| VLAN | Name        | Subnet           | Gateway        |
|------|------------|------------------|---------------|
| 10   | USERS      | 192.168.10.0/24  | 192.168.10.1  |
| 20   | ADMIN      | 192.168.20.0/24  | 192.168.20.1  |
| 30   | HR         | 192.168.30.0/24  | 192.168.30.1  |
| 40   | ACCOUNTING | 192.168.40.0/24  | 192.168.40.1  |
| N/A  | R1–R2 LINK | 10.0.12.0/30     | -             |
| N/A  | INTERNET   | DHCP (NAT)       | -             |

- VLAN segmentation across multiple switches  
- 802.1Q trunking between switches  
- Router-on-a-Stick inter-VLAN routing  
- DHCP for automatic IP assignment  
- NAT overload for internet access  
- Static routing between R1 and R2  
- ACL to restrict VLAN access  
---
## Management IPs
- SW1 VLAN 20: 192.168.20.4/24
- SW2 VLAN 20: 192.168.20.3/24
---
## Switch1
<img width="488" height="1143" alt="switch1" src="https://github.com/user-attachments/assets/bd2b4b38-30f2-43a6-8086-a09e9f0c15c0" />

Switch1 is the main access switch in the network. It is connecting end devices (PCs and Ubuntu) and forwards traffic to the router (Router1) for inter-VLAN Routing.
on switch1, I created multiple VLANs (10,20,30,40) to put different departments such as Users, Admim, HR and Accounting. Each device port was assigned to the correct VLAN to ensure proper network segmentation.

I configured trunk links between:
- Switch1 and Router1 this is for Router-on-a-Stick
- Switch and Switch2 this will allow VLAN traffic between switches
Switch1 is responible for:
- Separating traffic using VLANs
- Forwarding tagged traffic to the router for routing
- Allowing communication between switches through trunk links

overall, Switch1 acts as the central layer 2 device that organises network traffic and sends it to the router when communication between VLANs required.

---
## Switch2
<img width="548" height="1220" alt="switch2" src="https://github.com/user-attachments/assets/76209a40-ce60-4340-b8d7-c9eb19d2bdff" />

Switch2 acts as a secondary access switch connected to Switch1. It extends the network by allowing more devices to join different VLANs.
on Switch2, I configured the same VLANs (10,20,30,40) to match Switch1. Device ports were assigned to the correct VLANs so that end devices could communicate within their network segment.
A trunk link was configured between Switch 2 and Switch1, allowing VLAN traffic to pass between both switches. This ensures that devices on Switch2 can still reach the router (Route1) through Switch1 for inter-VLAN routing.

Switch2 is responsible for:
- Connecting additional end devices to the network
- Maintaining VLAN consistency across switches
- forwarding VLAN traffic to Switch1 via trunk
  
Overall, Switch2 will extend the network while maintaining proper VLAN segmentation and connectivity the rest of the infrastructure.

---
## Router1
### Sub-Interface
 <img width="340" height="1021" alt="subinterface" src="https://github.com/user-attachments/assets/863e9835-7c01-48c3-934b-c7b00c849f5b" />

Router1 is the main layer 3 device in the network and is responisible for routing traffic between VLANs. I configured sub-interface on FastEthernet0/0 using Router-on-a-Stick so that each VLAN has it own default gateway.
Each sub-interface was assigned to a specific VLAN using 802.1Q encapsulation and given an IP address to act as the gateway for the network. This allows devices in different VLANs to communicate with each other through Router1.
Router1 is also connected to Router2 using a separate point-to-point link, and a default route was added so that traffic can be forwarded towards the internet.

overall, this section handles:
- Inter-VLAN routing
- Default gateways for each VLAN
- Forwarding traffic towards Router2

### DHCP
<img width="285" height="244" alt="dhcp" src="https://github.com/user-attachments/assets/ea632634-e65d-4e67-96f3-36932c70df1c" />

on Router1, I configured DHCP pools for each VLAN so devices can automatically receive an IP address, subnet mask, and default gateway.\

Each VLAN has it own DHCP pool:
- VLAN 10 = USERS
- Vlan 20 = ADMIN
- Vlan 30 = HR
- vlan 40 = ACCOUNTING

This allow end devices to join the network without manual IP configuration. When a device connects to the network, Router1 assigns it an address based on the VLAN it belongs to.

Overall, this section handlers:
- Automatic IP address assignment
- Default gateway distribution
- Simple network management for end devices
  
---
## Router2
<img width="503" height="1010" alt="r2" src="https://github.com/user-attachments/assets/85573f5a-61e6-4051-ad75-8e26735d84ba" />

Router2 acts as the edge router that connects the internal network to the internet (via NAT). It sits between Router1 and the NAT cloud and is resonsible for forwarding traffic outside the network. 
I configured a point-to-point link between Router1 and Router2 using the 10.0.12.0/30 network. Router2 then uses NAT (overload) to translate internal private IP addresses (192.168.0.0) into public-facing address on its external interface.
An access list was created to define which internal networks are allowed to be translated, and NAT is applied so multiple inernal devices can share a single external IP.
A static route was also configured so Router2 knows how to reach the internal VLAN networks through Router1.

Overall, Router2 is responsible for:
- NAT (PAT) for internet access
- Translating private IPs to public IP
- Routing traffic between internal network and internet
- Acting as the network edge device

  
---
## Ubuntu
ACL - Run on Ubuntu

### Option 1
<img width="1104" height="1074" alt="option1" src="https://github.com/user-attachments/assets/ea821d05-cebd-46c8-acbb-722bc5d32322" />

This ACL policy is designed to restrict communication from HR VLAN (VLAN30) to the ADMIN VLAN (VLAN20) while allowing all other traffic.
I created an extended ACL name HR_BLOCK_ADMIN that denis traffic from the HR student (192.168.30.0/24) to the ADMIN subnet (192.168.20.0/24). After the deny rule, a permit ip any any statement is used to allow all other traffic to pass normally.
The ACL is applied inbound on the VLAN 30 sub-interface (FastEthernet0/0.30), which means traffic is filtered as its enters the router from the HR network.

This ensures:
- HR users cannot access ADMIN resources
- HR users can still access other VLANs and the internet


### ACL Testing
<img width="895" height="370" alt="pc1" src="https://github.com/user-attachments/assets/41719672-41f7-4c40-82f5-fa2fc8f91d83" />

Testing confirmed that the ACL was working as expected.

Devices in VLAN30 were unable to reach devices in VLAN 20, reciving a "communication administratively prohibited" response, which indicates the ACL is actively blocking the traffic.

At the same time, communication to other networks remained unaffected, confirmining that only the intended traffic was restricted.

This verifies that:
- VLAN 30 = VLAN20 is successfully blocked
- All other traffic flow normally


### Option 2
<img width="1040" height="1003" alt="2run" src="https://github.com/user-attachments/assets/bd0f7cde-aa3f-4362-8f95-a00e2a970236" />

This ACL policy is designed to restrict communication from the USERS VLAN (VLAN 10) to the ACCOUNTING VLAN (VLAN 40) while allowing all other traffic.
I created an extended ACL named USERS_BLOCK_ACCOUNTING that denies traffic from the USERS subnet (192.168.10.0/24) to the ACCOUNTING subnet (192.168.40.0/24). After the deny rule, a permit ip any any statement is used so that all other traffic can continue normally.
The ACL is applied inbound on the VLAN 10 sub-interface (FastEthernet0/0.10), which means traffic is filtered as it enters the router from the USERS network.

This ensures:
- Users in VLAN 10 cannot access Accounting resources
- Users in VLAN 10 can still access other permitted networks

### ACL Testing
<img width="888" height="385" alt="pc2" src="https://github.com/user-attachments/assets/28004be8-b11e-438a-8d1c-ad1465b3b0ea" />

ACL testing confirmed that the access control rule was working as expected. Devices in VLAN 10 were still able to communicate with other permitted VLANs, but traffic from VLAN 10 to VLAN 40 successfully blocked. This showed that the ACL was correclty applied to restrict only the targeted network traffic without affecting the rest of the network.

---
# Python Script

```python

from netmiko import ConnectHandler
from getpass import getpass


def wildcard_from_mask(mask: str) -> str:
    parts = mask.split(".")
    wildcard = [str(255 - int(part)) for part in parts]
    return ".".join(wildcard)


def build_acl_config(
    acl_name: str,
    src_net: str,
    src_mask: str,
    dst_net: str,
    dst_mask: str,
    interface: str,
    direction: str = "in",
) -> list[str]:
    src_wc = wildcard_from_mask(src_mask)
    dst_wc = wildcard_from_mask(dst_mask)

    return [
        f"ip access-list extended {acl_name}",
        f"deny ip {src_net} {src_wc} {dst_net} {dst_wc}",
        "permit ip any any",
        "exit",
        f"interface {interface}",
        f"ip access-group {acl_name} {direction}",
    ]


def main() -> None:
    router = {
        "device_type": "cisco_ios",
        "host": input("Router IP: ").strip(),
        "username": input("Username: ").strip(),
        "password": getpass("Password: "),
        "secret": getpass("Enable secret: "),
    }

    print("\nChoose ACL policy:")
    print("1. Block HR VLAN 30 -> ADMIN VLAN 20")
    print("2. Block USERS VLAN 10 -> ACCOUNTING VLAN 40")
    choice = input("Enter choice: ").strip()

    if choice == "1":
        acl_name = "HR_BLOCK_ADMIN"
        src_net = "192.168.30.0"
        src_mask = "255.255.255.0"
        dst_net = "192.168.20.0"
        dst_mask = "255.255.255.0"
        interface = "FastEthernet0/0.30"
    elif choice == "2":
        acl_name = "USERS_BLOCK_ACCOUNTING"
        src_net = "192.168.10.0"
        src_mask = "255.255.255.0"
        dst_net = "192.168.40.0"
        dst_mask = "255.255.255.0"
        interface = "FastEthernet0/0.10"
    else:
        print("Invalid choice.")
        return

    commands = build_acl_config(
        acl_name=acl_name,
        src_net=src_net,
        src_mask=src_mask,
        dst_net=dst_net,
        dst_mask=dst_mask,
        interface=interface,
        direction="in",
    )

    try:
        with ConnectHandler(**router) as conn:
            conn.enable()
            output = conn.send_config_set(commands)
            print("\n=== CONFIG OUTPUT ===")
            print(output)

            print("\n=== ACL VERIFICATION ===")
            print(conn.send_command("show access-lists"))

            print("\n=== INTERFACE VERIFICATION ===")
            print(conn.send_command(f"show run interface {interface}"))

    except Exception as exc:
        print(f"Error: {exc}")


if __name__ == "__main__":
    main()

```
