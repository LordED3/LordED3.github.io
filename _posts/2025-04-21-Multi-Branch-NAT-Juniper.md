---
layout: post
title: Multi-Branch Network NAT with Juniper
date: 2025-04-21 01:10:00 -0500
categories: [Mini-Project]
tags: [Networking, OPNsense]
image:
  path: assets/images/thumbnails/NAT_OSPF Juniper.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Multi-Branch Enterprise Network Design with Juniper

## Overview

This project builds a simulated multi-branch enterprise network using Juniper vSRX routers and firewalls. A central HQ connects to two remote branches (Branch A and Branch B) via point-to-point links, with the HQ providing centralized internet access through NAT. Dynamic routing is achieved using OSPF to ensure seamless communication across all sites.
The lab focuses on key enterprise networking concepts, including dynamic routing, NAT/PAT and inter-VLAN routing.

## Project Objectives

- Configure OSPF Area 0 across all routers  
- Implement inter-VLAN routing at each branch  
- Establish security zones at HQ to separate internal and external networks  
- Set up NAT at HQ for internet access  
- Ensure full connectivity between branches and the internet  

## Topology

![Desktop View](assets/images/posts/2025-04-21-Multi-Branch-NAT-Juniper/OSPF_NAT_Juniper.png){: w="900" h="900" }
_Network Topology_

## Network Components

- **HQ Router/Firewall**: Juniper vSRX (flow-based mode)  
- **Branch A Router**: Juniper vSRX (packet-based mode)  
- **Branch B Router**: Juniper vSRX (packet-based mode)  
- **L2 Switches**: For VLAN segmentation at each branch  
- **ISP node**: Provides internet connectivity  

## IP Addressing Scheme

| Device     | Interface/Description     | IP Address / Subnet     | VLAN ID |
|------------|----------------------------|--------------------------|---------|
| HQ-R1-NAT  | Ge-0/0/0 – ISP int         | 10.0.137.100/32          |         |
|            | Ge-0/0/1 – BR-A-R1 int     | 10.0.0.9/30              |         |
|            | Ge-0/0/2 – BR-B-R1 int     | 10.0.0.5/30              |         |
| BR-A-R1    | Ge-0/0/0 – BR-B-R1 int     | 10.0.0.1/30              |         |
|            | Ge-0/0/1 – HQ-R1 int       | 10.0.0.10/30             |         |
|            | Ge-0/0/2.10 – VLAN 10 int  | 192.168.10.1/24          | 10      |
|            | Ge-0/0/2.20 – VLAN 20 int  | 192.168.20.1/24          | 20      |
| BR-B-R1    | Ge-0/0/0 – BR-A-R1 int     | 10.0.0.2/30              |         |
|            | Ge-0/0/2 – HQ-R1 int       | 10.0.0.6/30              |         |
|            | Ge-0/0/1.10 – VLAN 10 int  | 172.16.10.1/24           | 10      |
|            | Ge-0/0/1.20 – VLAN 20 int  | 172.16.20.1/24           | 20      |
| BR-A-Sw1   | G0/0 – Trunk               |                          | 10,20   |
| BR-B-Sw2   | G0/0 – Trunk               |                          | 10,20   |

## Configuration Summary

| Device     | Configuration Details                                       |
|------------|-------------------------------------------------------------|
| HQ-R1-NAT  | Interface assign, NAT/PAT, OSPF, Static route, Security Zones / policy, Routing Policy (distribute static route via OSPF) |
| BR-A-R1    | Interface assign, VLANs, OSPF                                |
| BR-B-R1    | Interface assign, VLANs, OSPF                                |
| BR-A-Sw1   | VLANs, trunking, access ports                                |
| BR-B-Sw2   | VLANs, trunking, access ports                                |
| VPCs       | Basic IP for testing NAT and reachability                    |

## HQ-R1-NAT Security Configs

> Note: This config allows any connection from WAN to LAN and vice versa for testing purposes.
{: .prompt-info }

```bash
set security nat source pool PUB-IP address 10.0.137.100/32
set security nat source rule-set NAT-to-INTERNET from zone TRUST
set security nat source rule-set NAT-to-INTERNET to zone UNTRUST
set security nat source rule-set NAT-to-INTERNET rule NAT-BRANCHES match source-address 192.168.10.0/24
set security nat source rule-set NAT-to-INTERNET rule NAT-BRANCHES match source-address 192.168.20.0/24
set security nat source rule-set NAT-to-INTERNET rule NAT-BRANCHES match source-address 172.16.10.0/24
set security nat source rule-set NAT-to-INTERNET rule NAT-BRANCHES match source-address 172.16.20.0/24
set security nat source rule-set NAT-to-INTERNET rule NAT-BRANCHES then source-nat pool PUB-IP

set security policies from-zone TRUST to-zone UNTRUST policy WAN-Internet match source-address any
set security policies from-zone TRUST to-zone UNTRUST policy WAN-Internet match destination-address any
set security policies from-zone TRUST to-zone UNTRUST policy WAN-Internet match application any
set security policies from-zone TRUST to-zone UNTRUST policy WAN-Internet then permit

set security policies from-zone UNTRUST to-zone TRUST policy Internet-WAN match source-address any
set security policies from-zone UNTRUST to-zone TRUST policy Internet-WAN match destination-address any
set security policies from-zone UNTRUST to-zone TRUST policy Internet-WAN match application any
set security policies from-zone UNTRUST to-zone TRUST policy Internet-WAN then permit

set security zones security-zone UNTRUST host-inbound-traffic system-services all
set security zones security-zone UNTRUST host-inbound-traffic protocols all
set security zones security-zone UNTRUST interfaces ge-0/0/0.0

set security zones security-zone TRUST address-book address BRANCH-A 192.168.10.0/24
set security zones security-zone TRUST address-book address BRANCH-B 172.16.10.0/24
set security zones security-zone TRUST address-book address BRANCH-B-2 172.16.20.0/24
set security zones security-zone TRUST address-book address BRANCH-A-2 192.168.20.0/24
set security zones security-zone TRUST address-book address-set VLAN-BRANCHES address BRANCH-A
set security zones security-zone TRUST address-book address-set VLAN-BRANCHES address BRANCH-B
set security zones security-zone TRUST address-book address-set VLAN-BRANCHES address BRANCH-A-2
set security zones security-zone TRUST address-book address-set VLAN-BRANCHES address BRANCH-B-2

set security zones security-zone TRUST host-inbound-traffic system-services all
set security zones security-zone TRUST host-inbound-traffic protocols all
set security zones security-zone TRUST interfaces ge-0/0/1.0
set security zones security-zone TRUST interfaces ge-0/0/2.0
```

## Testing and Validation
The following tests were performed to validate the configuration. The demo video below showcases these tests.

- **OSPF Neighbors**: Verification of OSPF neighbor relationships using `show ospf neighbor`.  
- **Inter-VLAN Communication**: Connectivity test between VLANs within each branch.  
- **Inter-Branch Communication**: Connectivity test between devices in different branches.  
- **Internet Access**: Successfull pings to an external IP (e.g., ping 1.1.1.1) from branch devices via NAT.  

## 🎥 Demo Video

The video below showcases:

- NAT functionality 
- Successful Inter-VLAN and Inter-branch routing.
- OSPF neighbor establishment and route learning

{% include embed/youtube.html id='brzFmtPtWwo?si=rIf4shtSzdd-I2hZ' %}

## 📂 Final Configurations:
All the final configuration files for each device are available [Config Files.](https://github.com/LordED3/OSPF-NAT-Juniper-configs./tree/main)

## Conclusion
This project demonstrates essential enterprise networking practices using Juniper vSRX devices. The successful deployment of OSPF, NAT, and VLANs ensures robust connectivity across branches and lays the groundwork for exploring more advanced topics like BGP, stricter firewall policies, and DMZ deployment.
