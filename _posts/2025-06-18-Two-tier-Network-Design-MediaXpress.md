---
layout: post
title: Two-tier Network Design MediaXpress
date: 2025-06-18 01:10:00 -0500
categories: [Project]
tags: [Networking, OPNsense]
image:
  path: assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/2-tier.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---
# Two-Tier Network Design and Implementation for MediaXpress

## Overview

### Company Background
**MediaXpress** is a mid-sized media production company specializing in commercial video production, print media, and digital content marketing. 
The company's headquarters (HQ), located in a Toronto, hosts centralized IT infrastructure including Active Directory, file servers, VoIP services, and high-performance design workstations.

This project outlines my proposed solution to address the company’s growing IT requirements. I have focused on designing and implementing a two-tier network architecture that meets MediaXpress's needs for scalability, security, and high performance. The goal of this solution is to create a robust and manageable network infrastructure that will support the company’s diverse operations both now and in the future.

## Key Network Requirements for MediaXpress
The network design for MediaXpress must meet several key requirements to ensure optimal performance, security, and scalability. The following requirements will guide the network architecture and its implementation:
1.	**Network Redundancy**
To ensure continuous availability and minimize the risk of downtime, the network must include redundancy at critical points. This will involve having backup links, redundant hardware, and failover mechanisms to ensure business continuity even in the event of hardware failures or network issues.

2.	**Network Separation via VLANs**
Given the diverse range of activities at MediaXpress (video production, digital marketing, print media, etc.), the network must be segmented into multiple Virtual Local Area Networks (VLANs). This will enhance security, improve network performance, and allow for better management of traffic flows between different departments and user groups. Each department (e.g., marketing, production, administration) will be isolated into its own VLAN.

3.	**Scalability for 300–500 Employees, with Potential to Expand to 1000+ Employees**
The design must accommodate the current employee base of 300–500 employees, with the flexibility to scale to over 1000 employees in the future. The network architecture should be built with scalability in mind to support rapid growth in both the number of users and the increasing demand for bandwidth as the company expands.

4.	**Ease of Network Management and Monitoring**
The network should be easy to manage, configure, and monitor. This includes centralized management tools, automated configuration, and real-time network monitoring capabilities to ensure high performance and quick issue resolution. A simple and intuitive interface will help network administrators maintain smooth operations without a steep learning curve.

5.	**Integration with Active Directory Services**
Active Directory (AD) services will be central to the network's identity management, user authentication, and resource access control. The network design must include proper integration with AD to support centralized user authentication, group policies, and resource access control for both employees and contractors.

## Network Design Overview

The network solution proposed is based on a two-tier architecture, which provides the ideal balance of scalability, performance, and redundancy to meet the current and future needs of MediaXpress. The two-tier design includes:

**Edge Router/Security:** An OPNsense firewall acts as the internet edge firewall / router device

**Core Layer:** Dual Layer 3 core switches (C-SW-A, C-SW-B) provide redundancy and inter-VLAN routing.

**Access Layer:** Four access Layer 2 switches (AC-SW-1 through AC-SW-4) support user and endpoint connectivity.

**VLAN Structure:** The VLANs have been allocated based on MediaXpress's different departments and needs:

| VLAN ID | Department           |
|---------|----------------------|
| 10      | Server Room          |
| 20      | Production           |
| 30      | Design               |
| 40      | Finance/HR           |
| 50      | Marketing & Sales    |
| 60      | Voice                |
| 80      | Logistics            |
| 100     | Management/IT        |


**Routing:** Inter-VLAN routing is handled by the core L3 switches while External routing, NAT and firewall functions are managed by the OPNsense firewall.

## Network Topology
The topology below illustrates how the core, access, and edge layers are interconnected to provide a resilient, scalable, and secure network.

![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/Media Company Low level overview - HQ.png){: w="900" h="900" }
_Network Topology_

## IP Addressing table
This table defines the IP address assignments for each device, VLAN, and subnet in the network, ensuring proper communication, segmentation, and security.

| Device              | Interface / Description                                     | IP address / Subnet    | Gateway                                | VLAN ID |
|---------------------|-------------------------------------------------------------|------------------------|----------------------------------------|---------|
| **Toronto Headquarters** |                                                             |                        |                                        |         |
| HQ-FW-R (OPNSense)  | Eth 0 – WAN                                                 | Auto Config.           |                                        |         |
|                     | Eth 1 – MGT                                                 | 192.168.1.1/24         |                                        |         |
|                     | Eth 2 – link L3-SW-A                                        | 10.0.0.5/30            |                                        |         |
|                     | Eth 3 – link L3-SW-B                                        | 10.0.0.9/30            |                                        |         |
| HQ-CSW-A            | G0/0 – link FW-A                                            | 10.0.0.6/30            |                                        |         |
|                     | G0/1 + G0/2 – link HQ-CSW-B LAGG 1 or port-channel 1        | 10.0.0.1/30            |                                        |         |
|                     | VLAN 10 - SVI                                               | 192.168.10.2/24        | 192.168.10.1/24 - HSRP                | 10      |
|                     | VLAN 20 - SVI                                               | 192.168.20.2/24        | 192.168.20.1/24 - HSRP                | 20      |
|                     | VLAN 30 - SVI                                               | 192.168.30.2/24        | 192.168.30.1/24 - HSRP                | 30      |
|                     | VLAN 40 - SVI                                               | 192.168.40.2/24        | 192.168.40.1/24 - HSRP                | 40      |
|                     | VLAN 50 - SVI                                               | 192.168.50.2/24        | 192.168.50.1/24 - HSRP                | 50      |
|                     | VLAN 60 - SVI                                               | 192.168.60.2/24        | 192.168.60.1/24 - HSRP                | 60      |
|                     | VLAN 80 - SVI                                               | 192.168.80.2/24        | 192.168.80.1/24 - HSRP                | 80      |
|                     | VLAN 100 - SVI                                              | 192.168.100.2/24       | 192.168.100.1/24 - HSRP               | 100     |
| HQ-CSW-B            | G0/0 – link FW-A                                            | 10.0.0.10/30           |                                        |         |
|                     | G0/1 + G0/2 – link HQ-CSW-A LAGG 1 or port-channel 1        | 10.0.0.2/30            |                                        |         |
|                     | VLAN 10 - SVI                                               | 192.168.10.3/24        | 192.168.10.1/24 - HSRP                | 10      |
|                     | VLAN 20 - SVI                                               | 192.168.20.3/24        | 192.168.20.1/24 - HSRP                | 20      |
|                     | VLAN 30 - SVI                                               | 192.168.30.3/24        | 192.168.30.1/24 - HSRP                | 30      |
|                     | VLAN 40 - SVI                                               | 192.168.40.3/24        | 192.168.40.1/24 - HSRP                | 40      |
|                     | VLAN 50 - SVI                                               | 192.168.50.3/24        | 192.168.50.1/24 - HSRP                | 50      |
|                     | VLAN 60 - SVI                                               | 192.168.60.3/24        | 192.168.60.1/24 - HSRP                | 60      |
|                     | VLAN 80 - SVI                                               | 192.168.80.3/24        | 192.168.80.1/24 - HSRP                | 80      |
|                     | VLAN 100 - SVI                                              | 192.168.100.3/24       | 192.168.100.1/24 - HSRP               | 100     |
| HQ-AC-SW-1          | G0/0 – 1 - Trunk link                                       |                        |                                        |         |
|                     | VLAN 100 - SVI                                              | 192.168.100.4/24       | 192.168.100.1/24 - HSRP               | 100     |
| HQ-AC-SW-2          | G0/0 – 1 - Trunk link                                       |                        |                                        |         |
|                     | VLAN 100 - SVI                                              | 192.168.100.5/24       | 192.168.100.1/24 - HSRP               | 100     |
| HQ-AC-SW-3          | G0/0 – 1 - Trunk link                                       |                        |                                        |         |
|                     | VLAN 100 - SVI                                              | 192.168.100.6/24       | 192.168.100.1/24 - HSRP               | 100     |
| HQ-AC-SW-4          | G0/0 – 1 - Trunk link                                       |                        |                                        |         |
|                     | VLAN 100 - SVI                                              | 192.168.100.7/24       | 192.168.100.1/24 - HSRP               | 100     |

## Network Implementation and Configuration

I have implemented the network using EVE-NG (Emulated Virtual Environment Next Generation), which will allow for a comprehensive testing and validation of the design. Below is a table that summarizes each network device, the roles they play, and the essential configurations required to bring the network live.

| Device                    | Role                        | Key Configurations                                                                 | Additional Notes                                    |
|---------------------------|-----------------------------|------------------------------------------------------------------------------------|----------------------------------------------------|
| **Core Switch A (C-SW-A)** | Primary Core Switch         | 1. Configure VLANs (10-100) <br> 2. Set up HSRP for redundancy <br> 3. Enable routing & OSPF <br> 4. Enable DHCP relay <br> 5. Enable SSH and SNMP <br> 6. Enable STP Rapid PVST+ | Handles inter-VLAN routing & HSRP.                |
| **Core Switch B (C-SW-B)** | Backup Core Switch          | 1. Configure identical VLANs <br> 2. Set up HSRP <br> 3. Enable routing & OSPF <br> 4. Enable DHCP relay <br> 5. Enable SSH and SNMP <br> 6. Enable STP Rapid PVST+ | Failover for C-SW-A.                               |
| **Access Switch 1 (AC-SW-1)** | Access Switch (Trunk to Core) | 1. Configure trunk ports <br> 2. Assign VLANs <br> 3. Enable STP <br> 4. Enable SSH and SNMP <br> 5. Enable STP Rapid PVST+ | Provides user connectivity.                        |
| **Access Switch 2 (AC-SW-2)** | Access Switch (Trunk to Core) | Same as AC-SW-1                                                                     | Same configuration as AC-SW-1.                     |
| **Access Switch 3 (AC-SW-3)** | Access Switch (Trunk to Core) | Same as AC-SW-1                                                                     | Same configuration as AC-SW-1.                     |
| **Access Switch 4 (AC-SW-4)** | Access Switch (Trunk to Core) | Same as AC-SW-1                                                                     | Same configuration as AC-SW-1.                     |
| **OPNsense Firewall (HQ-FW-A)** | Edge Router/Security         | 1. Configure WAN interface <br> 2. Set up OSPF area 0 <br> 3. Configure NAT <br> 4. Set up firewall rules Alias, Floating rules, etc. | Manages routing & firewall for internet access.    |


I have linked the configuration files for each device below.
- HQ-FW-A 
- HQ-CSW-A
- HQ-CSW-B
- AC-SW-1
- AC-SW-2
- AC-SW-3
- AC-SW-4
- ISC DHCP

## Why the OPNsense Firewall Is Critical
The OPNsense firewall is the point of contact for all incoming and outgoing traffic. It plays a dual role as both the network’s edge router and primary security device.
The OPNsense configuration I implemented includes:
## Secure Interfaces
-	WAN interface (dynamic IP via DHCP, bogon filtering enabled)
-	LAN and dedicated point to point links to core switches (10.0.0.x /30 links)
-	Clear segmentation between, internal and external zones
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/Interfaces Overview.png){: w="900" h="900" }
_Network Interfaces overiew_


## Firewall Rules
-	Aliases to group networks, Ips etc.
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/Aliases.png){: w="900" h="900" }
_Firewall Aliases_
-	Floating rules allowing OSPF communications between the firewall and core switches
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/Floating Rules.png){: w="900" h="900" }
_floating Rules_
-	Allow rules for internal VLANs to route outbound traffic via NAT

## NAT Configuration
-	Hybrid mode for outbound NAT to allow both automatic and manual rules
-	Specific rules that NAT internal VLAN traffic (e.g., 192.168.10.0/24–192.168.100.0/24) to the WAN
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/Hybrid-NAT.png){: w="900" h="900" }
_Firewall NAT Outbound_

## Routing and High Availability
-	OSPF enabled with the core switches for dynamic routing
-	Priority configurations on OSPF interfaces for predictable routing behavior
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/OSPF RT.png){: w="900" h="900" }
_Routing Diagnostics: OSPF 1_
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/OSPF RT - 1.png){: w="900" h="900" }
_Routing Diagnostics: OSPF 2_
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/OSPF RT - 2.png){: w="900" h="900" }
_OSPF Neighbors_

## Additional Devices for Management and Maintenance
Beyond the core networking components, I’ve integrated several additional systems into MediaXpress network. These enhance manageability, monitoring, and centralized services, helping the IT team maintain a healthy and secure environment while preparing the network for future growth.

## DHCP Server
A dedicated ISC DHCP server has been deployed to handle DHCP services across the network. While the Layer 3 core switches are configured as DHCP relays, the Ubuntu DHCP server provides:
-	Centralized management of IP address assignments
-	Flexibility for creating and updating DHCP scopes
-	Easier integration with DNS and other services
-	Logging of lease assignments for audit and troubleshooting purposes

![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/ISC-DHCP Running.png){: w="900" h="900" }
_ubuntu DHCP server_

This setup ensures consistency in IP management across all VLANs and simplifies future changes or expansions.

## LibreNMS Logging and Monitoring Server
To provide real-time visibility into the health of the network, I’ve added a LibreNMS server. LibreNMS offers:
-	Comprehensive network device monitoring using SNMP and syslog
-	Performance graphs and historical data to track trends and identify bottlenecks
-	Alerting for key events (e.g., interface down, high CPU, excessive traffic)
-	Integration potential with email, Slack, and other alerting systems

![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/LibreNMS - Device overview.png){: w="900" h="900" }
_LibreNMS device overview_

![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/LibreNMS - Dashboard.png){: w="900" h="900" }
_LibreNMS dashboard_

![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/LibreNMS - Network Map.png){: w="900" h="900" }
_LibreNMS custom network map_

This allows the IT team to proactively address issues before they impact business operations, and helps ensure the network remains highly available and performant.

## Windows Server 2022 for Active Directory Domain Services
A Windows Server 2022 instance has been configured as the Active Directory Domain Controller for MediaXpress. This server handles:
-	Centralized authentication and authorization for users and devices
-	Group Policy management to enforce security and operational policies
-	DNS services for internal name resolution
-	Support for future features like certificate services, file shares and SCCM

Active Directory plays a vital role in enforcing identity-based security and ensuring that only authorized users and devices can access network resources.
These additional systems complement the network’s two-tier design and robust security model, creating a well-rounded infrastructure that is easy to manage, secure, and ready for scaling as MediaXpress grows.

## Testing and Validation
Once all devices were configured, I carried out comprehensive testing to validate the design, ensure stability, and confirm that the network meets all of MediaXpress’s requirements. The testing process covered:
-	Connectivity between VLANs and core services (e.g., AD, DHCP, VoIP)
-	Redundancy and failover confirming that core switch failover (HSRP) and OSPF dynamic routing function as intended
-	Firewall rules validation ensuring that the OPNsense firewall correctly allows authorized traffic and blocks unauthorized connections
-	NAT and internet access verifying that internal devices can access the internet via outbound NAT while remaining protected
-	Monitoring and alerting testing LibreNMS and Monitoring the network devices for alerts on critical thresholds like CPU, memory, interface down events, etc.

> There is a single point of failure at the ISP node but I am in a lab environment so it is acceptable for testing purposes.
{: .prompt-warning }

### **Demo Video**
A demo video is linked below, showcasing the entire project in action, including the network topology, configuration highlights, and validation tests. This demo provides a visual walkthrough of the setup and how each component interacts within the design.

> Note: This is a place holder for now it will be updated soon, thank you.
{: .prompt-info }
{% include embed/youtube.html id='brzFmtPtWwo?si=rIf4shtSzdd-I2hZ' %}

## Conclusion
This project successfully establishes the foundation of a robust two-tier network design for MediaXpress’s headquarters. The network delivers:
-	Strong security at the edge via OPNsense, with enforced segmentation and NAT
-	Reliable inter-VLAN routing through dual core switches with HSRP for redundancy
-	Scalability to accommodate future growth in users, devices, and services
-	Centralized monitoring via LibreNMS to support proactive network management

## Future Plans
This network design was created with expansion in mind. Planned future upgrades will build on this solid groundwork and include:
-	Adding new branch sites, securely connected via WireGuard VPN and fully integrated into the existing dynamic routing and security framework
![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/Media Company 2 Tier-Network Deskign Project - Overview.png){: w="600" h="600" }
_Ottawa Branch Expansion Diagram_

-	Deploying SCCM (System Center Configuration Manager) to automate application deployment, patch management, and asset inventory for production PCs

![Desktop View](assets/images/posts/2025-06-01-Two-tier-Network-Design-MediaXpress/SCCM Deployment Diagram.png){: w="400" h="400" }
_Ottawa Branch Expansion Diagram_
This project represents just the first step. It provides the necessary platform for these future enhancements, ensuring that MediaXpress’s IT infrastructure can continue to scale and adapt as business needs evolve.
