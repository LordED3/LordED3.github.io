---
layout: post
title: Deploying Software with MECM
date: 2024-02-27 01:10:00 -0500
categories: [Project]
tags: [IT]
image:
  path: assets/images/posts/2024-02-27-MECM/MECM_Image.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Microsoft Endpoint Configuration Manager

## Introduction

This project aims to demonstrate the implementation and utilization of Microsoft Endpoint Configuration Manager (MECM) for centralized management of client devices within an organization. By leveraging MECM, administrators can efficiently deploy software, manage updates, and automate routine tasks, thereby enhancing productivity and security across the network.

## Objectives

1. **Setup Active Directory and Users:**
   
2. **Install and Configure Microsoft Endpoint Configuration Manager:**

3. **Software Deployment and Patch Management:**

4. **Automating Tasks with PowerShell Scripts:**

## Tools
- VMware Workstation Pro
- 1 Windows Server VM (Active Directory Domain Controller)  --> IP address: 192.168.10.1 /24
- 1 Windows Server VM (Endpoint Configuration Manager) --> IP address: 192.168.10.2 /24
- 2 Windows 10 client VMs all on the same virtual network --> IP addresses: 192.168.10.10, 20 /24
- Network Adapters: NAT and Virtual-Network (VMnet2) on all devices.

**Note:** In realistic scenarios, IP addresses will be assigned by a dedicated DHCP server. Static IPs will become tedious with scale.

## Installing and Configuring Windows AD and Windows Domain Controller

I have deployed and configured a Windows Server 2022 with AD DNS and have created a domain named `divinehomelab.ca`. Within the domain, I have also created an organizational unit 'SCCM' with multiple users and groups in the OU. The users and groups that will be created in the OU are listed below:

- SCCMAdmin
- SQLSvrAgent
- SCCM Network Access 
- SCCM Client Push Install 
- SCCM Domain Join 
- SCCM Admin Group
- SCCM SQL Reporting

**Note:** Using multiple Windows servers with different roles instead of one is a good practice for realistic situations.

There is also a need to create a container called System Management before the delegation. Some delegation to the SCCM server will also be created.

## Installing Microsoft Endpoint Configuration Manager and Prerequisites on Windows Server

The MECM server will need to be connected to the domain controller, and prerequisites will also need to be installed before installing MECM. The prerequisites are listed below:

- Extend Windows AD schema on the DC
- SQL Server 2019 (Firewall Rules will be created for inbound management of SQL on the SCCM server)
- MS SQL Management Studio
- Windows ADK and Windows PE add-on
- SCCM 2203

**Firewall Inbound Rules on SCCM**

We are creating inbound rules for SQL Configuration Management on the SCCM server. Follow the steps in the screenshots below.

**Roles and Features Prerequisites**

The roles and feature prerequisites needed on the SCCM server are listed below. I have also written a PowerShell script to add the roles and features.

- IIS web server with all management tools, Application Development role, Performance role, Health and Diagnostic roles, Common HTTP features, Windows Server Update Services, .NET Framework 3.5, Remote Differential Compression.

```powershell
# Install IIS role with all management tools and features
Add-WindowsFeature -Name Web-Server -IncludeAllSubFeature

# Install Application Development role
Add-WindowsFeature -Name Web-App-Dev

# Install Performance role
Add-WindowsFeature -Name Web-Performance

# Install Health and Diagnostic roles
Add-WindowsFeature -Name Web-Health

# Install Common HTTP features
Add-WindowsFeature -Name Web-Http-Errors, Web-Http-Logging, Web-Http-Tracing, Web-Default-Doc, Web-Dir-Browsing, Web-Static-Content, Web-Http-Redirect

# Install Windows Server Update Services
Add-WindowsFeature -Name UpdateServices, UpdateServices-WidDB, UpdateServices-Services

# Install .NET Framework 3.5
Add-WindowsFeature -Name NET-Framework-Features

# Install Windows Remote Differential Compression
Add-WindowsFeature -Name FS-Remote-Differential-Compression
```

After installing the Roles and Features, we need to install:
-**SQL Server 2019 Standard**
The steps to install it are in the screenshots below.

After the installation is complete, we will use the account (SQLSvrAgent) created in the Domain Controller to configure the SQL Server 2019.

**Note:** Open SQL Server Management Studio and configure the memory to match the server specifications. In my case, I have 4GB RAM and have set it to the maximum server memory.

- MS SQL Server Management Studio

After installing and configuring SQL Server 2019, we will download and install SSMS. The installation process is in the screenshots below.

- Windows ADK

Finally, installing Windows ADK, the PE add-on is also needed to be installed. Both are installed similarly.

- Install and configuration of MECM 2303

Finally, after all the prerequisites have been installed, MECM will be installed and configured. After MECM is configured, we can start software deployment to clients.

## Using Endpoint Configuration Manager for Software Deployment and PowerShell Script Execution

- Demonstrate the process of deploying software applications to client devices using MECM's software deployment feature.
- Illustrate the steps for creating and deploying PowerShell scripts via MECM to automate routine tasks.
- Showcase the flexibility and efficiency of MECM in managing diverse IT environments and addressing operational challenges.

## Conclusion

In conclusion, this project has provided a comprehensive overview of Microsoft Endpoint Configuration Manager and its role in modern IT management. By successfully implementing MECM, organizations can centralize device management, streamline administrative tasks, and enhance overall security posture. Moving forward, continued exploration and utilization of MECM capabilities will enable organizations to adapt to evolving technological landscapes and meet the demands of an increasingly digital world.
