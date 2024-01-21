---
layout: post
title: Creating a honeypot in Azure
date: 2024-01-17 01:10:00 -0500
categories: [Blogging, Tutorial, Project]
tags: [cybersecurity, hacking, blueteam, devops]
image:
  path: assets/images/thumbnails/Honeypot2.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Creating an SSH Honeypot in the Cloud with Docker

## Introduction

This project aims to establish an SSH Honeypot in Azure using Docker, shedding light on the tactics employed by attackers who breach an SSH server. Specifically, we focus on automated brute-force attacks to extract behavioral information for strengthening security defenses.

### Topology
![Desktop View](assets/images/posts/2024-01-17-Honeypot-in-Azure/Honey Pot Topology croped.jpg){: width="900" height="900" }

## Tools

To execute this project, the following tools and resources are necessary:

- Desktop/laptop PC
- Cloud provider account (e.g., Azure)
- Docker/Docker Compose installed on the virtual machine (VM)
- Cowrie container

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
resource "azurerm_network_security_group" "HoneyPot_GRP_NSG" {
  name                = "honeypot-NSG"
  resource_group_name = azurerm_resource_group.HoneyPot_GRP.name
  location            = azurerm_resource_group.HoneyPot_GRP.location
}

# Define security rules for TCP and UDP ports
resource "azurerm_network_security_rule" "allow_tcp_ports" {
  name                        = "Allow_TCP"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  resource_group_name         = azurerm_resource_group.HoneyPot_GRP.name
  network_security_group_name = azurerm_network_security_group.HoneyPot_GRP_NSG.name

  source_address_prefix      = "*"  # Allow traffic from any source
  source_port_range          = "*"  # Allow traffic from any source port
  destination_address_prefix = "22, 69"  # Allow traffic to any destination
  destination_port_range     = "22, 69"  # Specify the TCP ports you want to allow
  protocol                   = "Tcp"
}

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
  size                            = "Standard_D2s_v3"
  admin_username                  = "azureuser"
  admin_password                  = var.admin_password
  disable_password_authentication = false


  network_interface_ids = [
    azurerm_network_interface.HoneyPot_GRP.id,
  ]



  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }


  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
}


```

## Step 2: Configuring the Honeypot Server

Once resources are in place, the Honeypot server is configured through the following steps:

0. **Changing SSH Port**
   - Safeguard existing SSH configuration: `sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup`.
   - Modify SSH configuration file: `sudo nano /etc/ssh/sshd_config`.
   - Change the SSH port I am changing mine from "Port 22" to "Port 69".
   - Restart SSH daemon: `sudo systemctl restart ssh`.

1. **Installing Docker and Docker Compose**
   - Update packages: `apt update && apt upgrade -y`.
   - Install Docker and Docker Compose: `sudo apt install docker.io docker-compose -y`.

2. **Creating a New User**
   - we will create a user cowrie for security reasons with: `addsuer cowrie` .
   - Establish a user for running the Cowrie container: `su - honeypot`.

3. **Pulling Cowrie Image**
   - Retrieve the Cowrie Docker image: `docker pull cowrie/cowrie:latest`.

4. **Creating Docker Compose File**
   - Develop a `docker-compose.yml` file with the provided configuration.

```yaml 
version: '3'
services:
  cowrie:
    image: cowrie/cowrie
    ports:
      - "2222:22"
      - "2223:23"
    volumes:
      - ./cowrie:/cowrie 
```

5. **Rerouting Traffic with iptables**
   - Redirect VM port 22 traffic to Cowrie container ports 2222 and 2223:
     ``` bash
     sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
     sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
     ```

With these steps completed, the Honeypot is ready to attract potential SSH attacks.

## Step 3: Analyzing Attack Logs

After the Honeypot has run for a period, logs can be analyzed to gain insights into attacker behavior. Locate logs at `./cowrie/log/tty/cowrie.json` to review attempted commands by potential attackers.

By following these steps, this project provides a deeper understanding of SSH attack patterns, contributing to enhanced security measures.
