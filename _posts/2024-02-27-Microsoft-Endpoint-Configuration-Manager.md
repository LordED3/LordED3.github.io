---
layout: post
title: Cloud MECM Lab Deployment
date: 2024-02-27 01:10:00 -0500
categories: [Project]
tags: [IT]
image:
  path: assets/images/posts/2024-02-27-MECM/MECM_Image.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Microsoft Endpoint Configuration Manager Deployment in Azure

## Introduction

This project is my attempt to automate the deployment and setup of Microsoft Endpoint Configuration Manager (MECM) on azure. MECM is a microsoft application for centralized management of client devices within an organization. Using MECM, administrators can efficiently deploy software, manage updates, and automate routine tasks, to multiple clients.thereby enhancing productivity and security across any organisation.

## Objectives

1. **Resoure Deployment with Terrafrom to Azure**

2. **Using Powershell to Setup Active Directory and Users**

3. **Using Powershell to Install Prequisites and download files**
   
4. **Install and Configure Microsoft Endpoint Configuration Manager**

5. **Demo Software Deployment and Patch Management to Client Computers**


## Tools
- Azure account
- 1 Windows Server VM (Active Directory Domain Controller)
- 1 Windows Server VM (Endpoint Configuration Manager)
- 1 Windows Server VM (Database)

## Resoure Deployment with Terrafrom to Azure

I have written a Terrafrom script to deploy the following:

- 3 windows 2022 Server
- 1 Resource Group
- 1 VNET(Virtual Network)
- 3 VNIC(Virtual Network Interface Cards)
- 1 NSG(Network Security Group)

```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "3.87.0"
    }
  }
}

provider "azurerm" {
  features {}
}


# Resource Group
resource "azurerm_resource_group" "MECM_Lab" {
  name     = "MECM"
  location = " eastus"
}

# VNET
resource "azurerm_virtual_network" "MECM_Lab" {
  name                = "MCEM_VNET"
  address_space       = ["10.0.0.0/24"]
  location            = azurerm_resource_group.MECM_Lab.location
  resource_group_name = azurerm_resource_group.MECM_Lab.name
}

# Public IP
resource "azurerm_public_ip" "MCEM_Lab" {
  name                = "MCEM_Lab_PublicIP"
  resource_group_name = azurerm_resource_group.MECM_Lab.name
  location            = azurerm_resource_group.MECM_Lab.location
  allocation_method   = "Dynamic" # Change to "Static" for a static public IP
}

# Create an NSG (Network Security Group)
resource "azurerm_network_security_group" "MECM_Lab" {
  name                = "MECM_NSG"
  location            = azurerm_resource_group.MECM_Lab.location
  resource_group_name = azurerm_resource_group.MECM_Lab.name
}

resource "azurerm_subnet" "MECM_Lab" {
  name                 = "internal_subnet"
  resource_group_name  = azurerm_resource_group.MECM_Lab.name
  virtual_network_name = azurerm_virtual_network.MECM_Lab.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "MECM_Lab" {
  count               = 3
  name                = "VM-nic"
  location            = azurerm_resource_group.MECM_Lab.location
  resource_group_name = azurerm_resource_group.MECM_Lab.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.MECM_Lab.id
    private_ip_address_allocation = "Dynamic"
    # public_ip_address_id          = azurerm_public_ip.MECM.id
  }
}

# Password Variable
variable "admin_password" {
  type        = string
  description = "Admin password for the virtual machine"
  sensitive   = true
}

# names for each VM
variable "vm_names" {
  type    = list(string)
  default = ["DC01", "MECM-Site", "Database"]
}

resource "azurerm_windows_virtual_machine" "MECM_Lab" {

  count               = length(var.vm_names) # it creates VMs according to the number of names in VM_names variable.
  name                = var.vm_names[count.index]
  resource_group_name = azurerm_resource_group.MECM_Lab.name
  location            = azurerm_resource_group.MECM_Lab.location
  size                = "Standard_F2"
  admin_username      = "adminuser"
  admin_password      = var.admin_password
  network_interface_ids = [
    azurerm_network_interface.MECM_Lab[count.index].id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }
}
```
>Using multiple Windows servers with different roles instead of one is a good practice for realistic situations.
{: .prompt-info }

## Active Directory and Users Setup

I have wrtten a powershell script to install ADDS, DNS on the AD server
the script will also create a domain `divinehomelab.ca`

```powershell
# Install Active Directory and reboot
Install-WindowsFeature ad-domain-services -IncludeAllSubfeature -IncludeManagementTools

# Wait for reboot to complete
Start-Sleep -Seconds 180  # Wait for 3 minutes

# Check if the computer was rebooted
$rebooted = $false
$maxWait = 300  # Maximum seconds to wait for the system to come back up
$waitTime = 0

while ($waitTime -lt $maxWait) {
    if (Test-Connection -ComputerName localhost -Quiet) {
        $rebooted = $true
        break
    }
    Start-Sleep -Seconds 10
    $waitTime += 10
}

if ($rebooted -and (Get-WindowsFeature -Name ad-domain-services).Installed) {
    # Create domain
    Install-ADDSForest -DomainName "divinehomelab.ca" -InstallDns

    # Create Organizational Unit
    New-ADOrganizationalUnit -Name "SCCM" -Path "DC=divinehomelab,DC=ca"
} else {
    Write-Host "Reboot or AD DS installation failed."
}

# download MECM 2303 to extend active directory Schema
$url = "https://go.microsoft.com/fwlink/p/?LinkID=2195628&clcid=0x409&culture=en-us&country=us"

$outputFile = "C:\MECM\MCM_Configmgr_2303.exe"  # path where the file is downloaded.

try {
    Write-Host "Downloading MECM 2303..."
    Invoke-WebRequest -Uri $url -OutFile $outputFile -ErrorAction Stop
    Write-Host "MECM 2303 downloaded successfully."
} catch {
    Write-Host "Failed to download MECM 2303: $_"
}

```
> This Powershell Script will be run on the VM named `DC01` there will be scripts to setup other servers.
{: .prompt-info }



## Conclusion

This Project is what I am currently working on I will update it as I continue to work on it Thank you.
