# SOHO-Network-Homelab
## Overview
This project documents the design and implementation of a secure SOHO network using VLAN segmentation, firewall rules, and a VPN for secure remote access.

Goals for this project:
  - Segmentation of network traffic using VLANs
  - Reduce attack surface using firewall rules to enforce inter-VLAN isolation
  - Maintaining secure remote access without exposing the internal network to the Internet

## Network Hardware
- AT&T Gateway with IP passthrough to ER605
- Router: TP-Link ER605
- Switch: NETGEAR Managed Switch (8 Ports)
- Wireless AP: TP-Link AX1800

## Network Topology
![Network Topology](https://github.com/user-attachments/assets/dc842a8a-8de3-42f4-959f-0340838ed508)

### VLAN Configuration
| VLAN | Subnet | DHCP | Notes |
| --- | --- | --- | --- |
| VLAN 10 (Mgmt) | 192.168.10.0/24 | Disabled | Router, switch, admin laptop |
| VLAN 20 (LAN)| 192.168.20.0/24 | 192.168.20.50-100 | PCs, printers, wireless devices |
| VLAN 30 (IoT) | 192.168.30.0/24 | 192.168.30.50-100 | Ring doorbell, cameras, IoT Hub |

- The router is given a static IP of 192.168.10.1 and the switch is given a static IP of 192.168.10.2
* Lower IP ranges are resvered for network devices for future upgrades
### Router Port Configuration
<img width="1023" height="333" alt="Router Port Configuration" src="https://github.com/user-attachments/assets/5bc338ba-2196-485d-b4d3-29768dd69030" />

- Port 1 is configured as the WAN interface 
- Port 2 is a trunk port connecting to the Switch and carries VLANs 10,20, and 30 as 802.1Q tagged traffic. 
- Ports 3-4 are also configured as trunk ports to support future VLAN supported devices. 
- Port 5 is an access port assigned to port 10 and will be used for my admin PC, allowing untagged traffic.
### Switch Port Configuration
<img width="525" height="683" alt="Switch Port Configuration" src="https://github.com/user-attachments/assets/3ed8b213-d7ba-491f-b9ff-a6f035412722" />

The managed switch is configured to support VLAN segmentation using 802.1Q. 802.1Q allows multiple VLANs to be carried over a single link.
### Trunk Port
- Port 1 - Uplink to Router
  - Connected to ER605 LAN Port 2
  - VLANs 10 (Mgmt), 20 (LAN), and 30 (IoT) are tagged
### Access Ports
- Port 2 - Access (VLAN 10 Mgmt)
  - Admin Laptop 192.168.10.5
  - Untagged access for admin tasks
- Port 3 - Access (VLAN 20 LAN)
  - Network Printer 192.168.20.50
- Port 4 - Access (VLAN 20 LAN)
  - Office PC 192.168.20.51
- Port 5 - Access (VLAN 20 LAN)
  - PC 2 192.168.20.52
- Port 6 - Access (VLAN 20 LAN)
  - AX1800 Acess Point 192.168.20.2
- Port 7 - Access (VLAN 30 IoT)
  - EUFY Server 192.168.30.2
  - Isolated from other VLANs for security
## Firewall Rules & Traffic Flow
<img width="1245" height="476" alt="Firewall Configuration" src="https://github.com/user-attachments/assets/436e09b2-7fc5-4171-bc15-9d06383d7f68" />

1. Management VLAN traffic is allowed to access all VLAN networks for admin purposes. 2. All non-managementt VLANs are explicitly blocked from accessing the Management VLAN.
3. The IoT VLAN is isolated and cannot communicate with any other VLAN networks.
4. Internet access is explicitly allowed for Management, LAN, and IoT VLANs.

## WireGuard VPN Implementation
WireGuard runs directly on the TP-Link ER 605. The security benefits of having a VPN include
  - End to end encyrption
  - No exposure of internal services to the Internet
  - VPN access is controlled by existing firewall policies
  - Management infrastructure remains unreachable 
### VPN Design
<img width="802" height="510" alt="WireGuard Configuration" src="https://github.com/user-attachments/assets/a37cdc53-735a-44ac-97ef-314d7c6ccaba" />
  
  - VPN clients are assigned IP addresses from the subnet: 10.6.0.0./24
  - WireGuard interface uses 10.6.0.1
The VPN traffic is then routed by the ER605 and is logically a part of VLAN 20 (LAN). Access to the Management VLAN (10) remains blocked as VPN clietnts inheirt the same policies as LAN devices in VLAN 10. This will allow VPN clients to function as if they were locally connected to the LAN VLAN (20).

### Split Tunneling
- Only internal subnet is routed through the VPN.
    - ex. 192.169.20.0/24
- Internet traffic will continue to use the VPN's client local internet connection

### Mobile File Access 
I've used the IPhone built in Files app and used WireGuard to connect back to the home network to access shared files hosted on my home PC. No external port forwarding or public file sharing services are required
  
<img width="1962" height="1589" alt="File Access with IPhone" src="https://github.com/user-attachments/assets/8cdce39e-401e-41de-bec6-f6e5bcf4db43" />

- The 'D" folder is the D: 


