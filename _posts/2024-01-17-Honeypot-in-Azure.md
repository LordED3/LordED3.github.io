---
layout: post
title: Deploying a honeypot in Azure
date: 2024-01-17 01:10:00 -0500
categories: [Project]
tags: [cybersecurity, blueteam, devops]
image:
  path: assets/images/thumbnails/Honeypot2.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Deploying an SSH Honeypot in the Cloud

## Introduction

This project aims to establish a Honeypot in Azure using Docker, shedding light on the tactics employed by attackers. Specifically, we focus on automated brute-force attacks through bots to extract behavioral information for strengthening security defenses.

### Topology
![Desktop View](assets/images/posts/2024-01-17-Honeypot-in-Azure/Honey Pot Topology - Updated.jpg){: w="900" h="900" }
_TPOT Container Topology_

## Tools

To execute this project, the following tools and resources are used:

- Desktop/laptop PC
- Cloud provider account (e.g., Azure)
- Cloud virtual machine (VM)
- [TPOT by TeleKom Security](https://github.com/telekom-security/tpotce)

## Step 1: Automating Resource Deployment with Terraform

To streamline the creation of Azure resources, Terraform is employed. The deployment process is as follows:

- Confirm Azure CLI login :`az login`.
- Validate account details :`az account show`.
- Initialize Terraform :`terraform init`.
- Preview resource creation :`terraform plan`.
- Apply the configuration :`terraform apply`.

Please Refer to the provided Terraform code below for resource deployment.

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


# create the Resource Group
resource "azurerm_resource_group" "HoneyPot_GRP" {
  name     = "HoneyPot"
  location = " eastus"
}

# Create VNET
resource "azurerm_virtual_network" "Pot" {
  name                = "Honeypot_VNET"
  address_space       = ["10.0.0.0/24"]
  location            = azurerm_resource_group.HoneyPot_GRP.location
  resource_group_name = azurerm_resource_group.HoneyPot_GRP.name
}

#Public IP
resource "azurerm_public_ip" "HoneyPot_GRP" {
  name                = "Honeypot_PublicIP"
  resource_group_name = azurerm_resource_group.HoneyPot_GRP.name
  location            = azurerm_resource_group.HoneyPot_GRP.location
  allocation_method   = "Dynamic" # Change to "Static" for a static public IP
}

# Create an NSG (Network Security Group)
resource "azurerm_network_security_group" "honeypot-NSG" {
  name                = "honeypot-NSG"
  location            = azurerm_resource_group.HoneyPot_GRP.location
  resource_group_name = azurerm_resource_group.HoneyPot_GRP.name
}

# Security rule for SSH on port 22
resource "azurerm_network_security_rule" "allow_ssh_port_22" {
  name                        = "Allow_SSH_22"
  priority                    = 300
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.HoneyPot_GRP.name
  network_security_group_name = azurerm_network_security_group.honeypot-NSG.name
}

# Security rule for different honeypot ports
/* 
resource "azurerm_network_security_rule" "open_ports" {
  name                        = "Honepot_Ports"
  priority                    = 1000
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_ranges     = ["0-65535"] # Adjust port ranges as needed
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.HoneyPot_GRP.name
  network_security_group_name = azurerm_network_security_group.honeypot-NSG.name
}
*/


#subnet
resource "azurerm_subnet" "HoneyPot_GRP" {
  name                 = "Honeypot_subnet"
  virtual_network_name = azurerm_virtual_network.Pot.name
  resource_group_name  = azurerm_resource_group.HoneyPot_GRP.name
  address_prefixes     = ["10.0.0.0/24"]

}

# create an Network Card for the Honeypot VM you can add more with count = 3  # number of NICs
resource "azurerm_network_interface" "HoneyPot_GRP" {


  name                = "Honeypot_NIC"
  location            = azurerm_resource_group.HoneyPot_GRP.location
  resource_group_name = azurerm_resource_group.HoneyPot_GRP.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.HoneyPot_GRP.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.HoneyPot_GRP.id

  }
}

#Password Variable
variable "admin_password" {
  type        = string
  description = "Admin password for the virtual machine"
  sensitive   = true
}

# Creating the Honeypot VM(Ubuntu Server 22.04)
resource "azurerm_linux_virtual_machine" "HoneyPot_GRP" {

  # count               = 3  # number of VMs
  name                            = "The-POT"
  resource_group_name             = azurerm_resource_group.HoneyPot_GRP.name
  location                        = azurerm_resource_group.HoneyPot_GRP.location
  size                            = "Standard_D4s_v3"
  admin_username                  = "azureuser"
  admin_password                  = var.admin_password
  disable_password_authentication = false


  network_interface_ids = [
    azurerm_network_interface.HoneyPot_GRP.id,
  ]



  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
    disk_size_gb         = 128
  }


  source_image_reference {
    publisher = "debian"
    offer     = "debian-11"
    sku       = "11-gen2"
    version   = "latest"
  }
}

```

> The Terraorm code here deploys the Honeypot VM, Firewall Rules NIC, etc.
{: .prompt-info }

## Step 2: Configuring the Honeypot Server

Once resources are in place, the Honeypot server is configured through the following steps:

- **Updating and Installing Git on Debian**
   - Update repositories: `sudo apt update && sudo apt upgrade -y`.
   - Install Git: `sudo apt install git`.

- **Downloading and Installing TPOT**
   - Clone the TPOT repository: `git clone https://github.com/telekom-security/tpotce`.
   - Change directory to tpotce: `cd tpotce/iso/installer/`.
   - Run the install script: `sudo ./install.sh --type=user`

> There is a prompt to set the web interface username and password during installation and it will take a while to install.
{: .prompt-warning }

- **Connecting to the Honeypot Web Interface**
   - Connect to the web interface: `https://VM_Public_IP:64297`.
  

- **Important Ports**
   - The install script will make some changes to the VM to allow management of the honeypot. I have included all the important management ports in the table below.

| Port        | Protocol | Direction | Description                                                   |
| :---        | :---     | :---      | :---                                                          |
| 80, 443     | tcp      | outgoing  | T-Pot Management: Install, Updates, Logs (i.e. Debian, GitHub, DockerHub, PyPi, Sicherheitstacho, etc). |
| 64294       | tcp      | incoming  | T-Pot Management: Access to Cockpit                           |
| 64295       | tcp      | incoming  | T-Pot Management: Access to SSH                               |
| 64297       | tcp      | incoming  | T-Pot Management Access to NGINX reverse proxy                |

[This is found in the Readme](https://github.com/telekom-security/tpotce/blob/master/README.md)

## Step 3: Analyzing Attacks with the Honeypot Management tools

The TPOT honeypot comes with a set of management tools to analyze atacks, log commands used etc. most off the Management tools use the Username and Password set during installation below is a table of the management tools and thier logins it can be found in the [Readme.](https://github.com/telekom-security/tpotce/blob/master/README.md)

| Service             | Account Type | Username / Group | Description                                                             |
| :---                | :---         | :---             | :---                                                                    |
| SSH, Cockpit        | OS           | `tsec`           | On ISO based installations the user `tsec` is predefined.               |
| SSH, Cockpit        | OS           | `<os_username>`  | Any other installation, the `<username>` you chose during installation. |
| Nginx               | BasicAuth    | `<web_user>`     | `<web_user>` you chose during the installation of T-Pot.                |
| CyberChef           | BasicAuth    | `<web_user>`     | `<web_user>` you chose during the installation of T-Pot.                |
| Elasticvue          | BasicAuth    | `<web_user>`     | `<web_user>` you chose during the installation of T-Pot.                |
| Geoip Attack Map    | BasicAuth    | `<web_user>`     | `<web_user>` you chose during the installation of T-Pot.                |
| Spiderfoot          | BasicAuth    | `<web_user>`     | `<web_user>` you chose during the installation of T-Pot.                |
| T-Pot               | OS           | `tpot`           | `tpot` this user / group is always reserved by the T-Pot services.      |
| T-Pot Logs          | OS           | `tpotlogs`       | `tpotlogs` this group is always reserved by the T-Pot services.         |

## **Attack Maps and Behaviour Analysis**
Leveraging the capabilities of TPOT's advanced management tools, we immerse ourselves in a dynamic understanding of cyber threats. TPOT seamlessly integrates tools that not only empower us to visualize the origins of attacks but also provide a comprehensive overview of IPs, their associated regions, the nature of attacks, and the behaviors exhibited—such as the commands utilized by attackers and their attempted compromises.

This rich and insightful data is dynamically translated onto a live map, offering a real-time representation of ongoing attacks. The visual narrative unfolds below through a series of screenshots, encapsulating the essence of these live attack maps. This visual journey provides a vivid portrayal of the ever-evolving cybersecurity landscape, ensuring a proactive stance in the face of emerging threats.
