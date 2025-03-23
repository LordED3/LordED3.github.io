---
layout: post
title: 🛡️Securing My Home Network with OPNsense
date: 2025-02-15 01:10:00 -0500
categories: [Project]
tags: [Networking, OPNsense]
image:
  path: assets/images/thumbnails/Home Network.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---
## Securing My Home Network with OPNsense: A Personal and Technical Journey

## 🎯Introduction: Why I Chose OPNsense

Securing my home network had always been a priority, but I wanted more than just a basic router with minimal firewall capabilities, especially after building my first Proxmox server. I explored several options, from pfSense to Ubiquiti and even considered some commercial-grade solutions. While each had its strengths, I found myself drawn to OPNsense. The fact that it’s open-source, regularly updated, and packed with enterprise-grade features made it an easy choice. Unlike many commercial solutions, OPNsense offered a level of transparency and flexibility that fit perfectly with my home lab goals. Plus, its modern UI and powerful plugins gave me the freedom to customize my firewall exactly how I needed. 

Another major reason for choosing OPNsense was its feature-rich environment. With built-in IDS/IPS (Intrusion Detection and Prevention System), robust VPN support, and advanced traffic shaping, OPNsense provided all the tools I needed to fine-tune my network. As someone who enjoys getting hands-on with network configurations, I was eager to explore its potential. The flexibility to dive deeper into firewall management, experiment with VLAN segmentation, and optimize traffic policies made the decision even more appealing. I knew this would be a learning curve, but the idea of customizing my home network to such a granular level was too exciting to pass up.

I was also eager to gain practical, real-world experience by managing my own enterprise-grade firewall setup at home. I didn’t just want a plug-and-play solution, I wanted to design, build, and manage a network that was both secure and optimized for performance. So, with that in mind, I embarked on this journey with OPNsense.

## 🛠️ Prototyping in EVE-NG: Testing Before Building
Before committing to hardware, I decided to prototype the entire setup in EVE-NG. This allowed me to simulate the complete network architecture and experiment with VLANs, firewall rules, VPN configurations, and intrusion prevention. I wanted to ensure that my design was not only functional but also scalable and secure before deployment.

In my virtual lab, I created a logical design that segmented the network into multiple VLANs:

- VLAN 10: Main network for trusted devices and management.

- VLAN 20: Guest and untrusted devices, isolated for security.

- VLAN 30: IoT devices with no internet access but minimal internal communication.

- VLAN 50: A DMZ to host Proxmox VMs with externally accessible services
 

## 📝 Network Topology
The network topology I simulated in EVE-NG closely mirrors the diagram shown above.
![Desktop View](assets/images/thumbnails/Protorype_EVE_NG.jpg){: w="900" h="900" }
_Network Topology_

## ⚙️ Configuring and Fine-Tuning
Once I had the prototype virtual instance up and running, I configured the basic network settings. The table below shows the configurations required for each device.:

| **Device**               | **Configurations**                                |
|--------------------------|---------------------------------------------------|
| 🛡️ OPNsense Router/Firewall | Interface assignments,VLANs, DHCP, Firewall Rules, IDS, IPS              |
| 🔀 Switch                  | Trunk configuration, VLAN tagging, Port Assignment |
| 📡 Wireless Access Point   | Wireless VLANs, VLAN tagging for SSIDs, Security            |

### OPNsense:
The OPNsense configuration includes the following:

1. Interface Assignments.
- Assign WAN and LAN interfaces properly via CLI.
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/Incorrect_Int_Assigned.png){: w="600" h="600" }
_Incorrect Interface assigned_
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/Correct_Int_Assigned.png){: w="600" h="600" }
_Correct Interface assigned_
- change LAN interface ip to desirerd ip via web ui.
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/LAN_IP.png){: w="600" h="600" }
_LAN IP address changed_
- Create additional VLAN interfaces for network segmentation.
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/vlan_interfaces_created.png){: w="600" h="600" }
_vlan devices created_

2. VLAN Configuration.
-  Assign the VLAN interfaces via web ui.
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/vlan_int_assigned.png){: w="600" h="600" }
_VLAN Interfaces assinged_
- Enable DHCP for each VLAN
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/vlan_DHCP.png){: w="600" h="600" }
_per-vlan DHCP enabled_

3. Firewall Rules.
- Set up basic rules for isolation, service and WAN access per-vlan.
![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/vlan_firewall rules.png){: w="600" h="600" }
_per-vlan firewall rules_

> The firewall rules are cloned and almost the same accross all vlans except management and iot. IoT Vlan is blocked from the internet and Management vlan is allowed to ping all the vlans.
{: .prompt-tip }

### Switch Configuration.
The switch configuration is straightforward and primarily involves configuring trunks, VLANs, and access ports to ensure proper traffic segmentation. Here’s the Cisco IOS configuration I used during prototyping, which includes trunk and access port assignments, VLAN definitions, and interface configurations.:

```
!
! Last configuration change at 19:37:16 UTC Sun Mar 23 2025
!
version 15.2
service timestamps log datetime msec
no service password-encryption
!
hostname Switch
!
boot-start-marker
boot-end-marker
!
no aaa new-model
ip cef
no ipv6 cef
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface GigabitEthernet0/0
 no shutdown
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface GigabitEthernet0/1
 no shutdown
!
interface GigabitEthernet0/2
 no shutdown
 switchport access vlan 10
!
interface GigabitEthernet0/3
 no shutdown
 switchport access vlan 20
!
interface GigabitEthernet1/0
 no shutdown
 switchport access vlan 20
!
interface GigabitEthernet1/1
 no shutdown
 switchport access vlan 50
!
interface GigabitEthernet1/2
 no shutdown
 switchport access vlan 50
!
interface GigabitEthernet1/3
 no shutdown
 switchport access vlan 30
!
interface GigabitEthernet2/0
 no shutdown
!
interface GigabitEthernet2/1
 no shutdown
 switchport access vlan 10
!
interface GigabitEthernet2/2
 no shutdown
!
interface GigabitEthernet2/3
 no shutdown
!
interface GigabitEthernet3/0
 no shutdown
!
interface GigabitEthernet3/1
 no shutdown
!
interface GigabitEthernet3/2
 no shutdown
!
interface GigabitEthernet3/3
 no shutdown
!
interface Vlan10
 no shutdown
 description Management
!
interface Vlan20
 no shutdown
 description Guest (My_house)
!
interface Vlan30
 no shutdown
 description IoT
!
interface Vlan50
 no shutdown
 description DMZ
!
ip http server
ip http secure-server
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
line vty 0 4
 login
!

```
## 📦 Hardware Planning: Building the Real Setup

With the prototype validated, I have shifted my focus to sourcing hardware for the physical build. I’m currently in the process of purchasing and assembling the necessary components to bring this design to life. Here’s a sneak peek at what’s going into the build:

- Intel N100 Mini PC for OPNsense  
- Managed TP-Link Switch with VLAN Support 
- TP-Link Access point with VLAN support 
- Proxmox Cluster (VLAN aware) for VMs and containers  
- NAS for Backup & File Storage 

The physical layout includes an 18u open-frame rack with a 24 port keystone patch panel, switch, NAS, Proxmox cluster, and the OPNsense firewall neatly wired to ensure easy access and maintenance.

![Desktop View](assets/images/posts/2025-02-15-OPNsense-Home-Network/Rack.png){: w="600" h="600" }
_Physical Rack Diagram for future ref_

## 📈 Performance Results and Future Plans

Even though I’m still finalizing the hardware and testing some configurations, the performance results from my EVE-NG prototype have been promising. Traffic segmentation is working perfectly, latency is minimal, and I plan test VPN connectivity, IDS/IPS and port forwarding.

Looking ahead, I have a few exciting plans:

- Deploying the Physical Setup: Once all the hardware arrives, I’ll transition from the prototype to the real network.

- Exploring WireGuard: For faster, lightweight VPN connections.

- Implementing High Availability (HA): To ensure network redundancy.

- Switching to ZFS file system to enable snapshots and easy back-up and restore.

- Fine-Tuning Traffic Shaping: To optimize streaming, gaming, and other latency-sensitive applications.

## 📚 Resources
This journey wouldn’t have been possible without the wealth of resources I leaned on:

- [OPNsense](https://docs.opnsense.org/) Official Documentation provided detailed insights into firewall rules, VPN configuration, and best practices.

- YouTube channels like Lawrence Systems, Home Network Guy, etc. offered practical, step-by-step tutorials.

- Community forums and Reddit threads were invaluable for troubleshooting and fine-tuning.

## 🔗 Final Thoughts: Why OPNsense is a Game-Changer

Building and securing my home network with OPNsense has been a deeply rewarding experience. It’s not just about protecting my network—it’s about gaining a deeper understanding of enterprise-grade firewall technologies and applying that knowledge in a real-world environment. OPNsense has empowered me with the tools and flexibility I need to create a secure, high-performance network that I can fully control.

As I move forward with the physical build, I’m excited to take this project to the next level and continue fine-tuning my home network for maximum security and performance. If you’re looking to level up your home network game, OPNsense is definitely worth considering!