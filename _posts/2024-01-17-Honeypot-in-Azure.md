---
layout: post
title: Creating a honeypot in Azure
date: 2024-01-17 01:10:00 -0500
categories: [Project]
tags: [cybersecurity, hacking, blueteam, devops]
image:
  path: assets/images/thumbnails/Honeypot2.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Creating an SSH Honeypot in the Cloud with Docker

## Introduction

This project aims to establish an SSH Honeypot in Azure using Docker, shedding light on the tactics employed by attackers who breach an SSH server. Specifically, we focus on automated brute-force attacks through bots to extract behavioral information for strengthening security defenses.

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
resource "azurerm_network_security_group" "honeypot-NSG" {
  name                = "honeypot-NSG"
  location            = azurerm_resource_group.HoneyPot_GRP.location
  resource_group_name = azurerm_resource_group.HoneyPot_GRP.name
}

# Security rule for SSH on port 22
resource "azurerm_network_security_rule" "allow_ssh_port_22" {
  name                        = "Allow_SSH_22"
  priority                    = 1000
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

# Security rule for SSH on port 69
resource "azurerm_network_security_rule" "allow_ssh_port_69" {
  name                        = "Allow_SSH_69"
  priority                    = 1001
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "69"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.HoneyPot_GRP.name
  network_security_group_name = azurerm_network_security_group.honeypot-NSG.name
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

> The Terraorm code here deploys the Honeypot VM and its resource group, it also setup firewall rules for a new ssh port "69"
{: .prompt-info }

## Step 2: Configuring the Honeypot Server

Once resources are in place, the Honeypot server is configured through the following steps:

0. **Changing SSH Port**
   - back up the existing SSH configuration: `sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup`.
   - Modify SSH configuration file: `sudo nano /etc/ssh/sshd_config`.
   - Change the SSH port I am changing mine from "Port 22" to "Port 69".
   - Restart SSH daemon: `sudo systemctl restart ssh`.

1. **Installing Docker and Docker Compose**
   - Update packages: `sudo apt update && sudo apt upgrade -y`.
   - Install Docker and Docker Compose: `sudo apt install docker.io docker-compose -y`.
   - Add docker to the super users group with: `sudo usermod -aG docker azureuser` 

> to avoid typing sudo every time we want to run docker commands azureuser is the username here
{: .prompt-tip }


2. **Pulling Cowrie Image**
   - Retrieve the Cowrie Docker image: `docker pull stingar/cowrie:latest`.

4. **Creating Docker Compose File**
   - I Created a `docker-compose.yml` file with the provided configuration.

```yaml 
version: '3'
services:
  cowrie:
    image: stingar/cowrie:latest
    restart: always
    volumes:
      - configs:/etc/cowrie
    ports:
      - "22:2222"
      - "23:2223"
    env_file:
      - cowrie.env
volumes:
    configs:
```
   - run `docker-compose up -d` to run the docker compose template

[Cowrie Docker Documentation](https://communityhoneynetwork.readthedocs.io/en/stable/cowrie/)
> you can find cowrie stingar in docker hub 
{: .prompt-info }

With these steps completed, the Honeypot is ready to attract potential SSH attacks.

## Step 3: Analyzing Attack Logs

After the Honeypot has run for a period, logs can be analyzed to gain insights into attacker behavior. Locate logs at `./cowrie/log/tty/cowrie.json` to review attempted commands by potential attackers.

By following these steps, this project provides a deeper understanding of SSH attack patterns, contributing to enhanced security measures.
