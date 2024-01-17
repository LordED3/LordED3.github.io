---
layout: post
title: Creating a honeypot in Azure
date: 2024-01-17 01:10:00 -0500
categories: [Blogging, Tutorial, Project]
tags: [Cybersecurity, Hacking]
image:
  path: assets/images/thubnails/Honey Pot-diagram.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

# Creating an SSH Honeypot in the Cloud with Docker

## Introduction

This project aims to establish an SSH Honeypot in Azure using Docker, shedding light on the tactics employed by attackers who breach an SSH server. Specifically, we focus on automated brute-force attacks to extract behavioral information for strengthening security defenses.

### Topology
[Insert Topology Image Here]

## Tools

To execute this project, the following tools and resources are necessary:

- Desktop/laptop PC
- Cloud provider account (e.g., Azure)
- Docker/Docker Compose installed on the virtual machine (VM)
- Cowrie container

## Step 1: Automating Resource Deployment with Terraform

To streamline the creation of Azure resources, Terraform is employed. The deployment process is as follows:

- Confirm Azure CLI login (`az login`).
- Validate account details (`az account show`).
- Initialize Terraform (`terraform init`).
- Preview resource creation (`terraform plan`).
- Apply the configuration (`terraform apply`).

Refer to the provided Terraform code for resource deployment.

## Step 2: Configuring the Honeypot Server

Once resources are in place, the Honeypot server is configured through the following steps:

0. **Changing SSH Port**
   - Safeguard existing SSH configuration: `sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup`.
   - Modify SSH configuration file: `sudo nano /etc/ssh/sshd_config`.
   - Adjust SSH port (e.g., from "Port 22" to "Port 69").
   - Restart SSH daemon: `sudo systemctl restart ssh`.

1. **Installing Docker and Docker Compose**
   - Update packages: `apt update && apt upgrade -y`.
   - Install Docker and Docker Compose: `sudo apt install docker.io docker-compose -y`.

2. **Creating a New User**
   - Establish a user for running the Cowrie container: `su - honeypot`.

3. **Pulling Cowrie Image**
   - Retrieve the Cowrie Docker image: `docker pull cowrie/cowrie:latest`.

4. **Creating Docker Compose File**
   - Develop a `docker-compose.yml` file with the provided configuration.

5. **Rerouting Traffic with iptables**
   - Redirect VM port 22 traffic to Cowrie container ports 2222 and 2223:
     ```
     sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
     sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
     ```

With these steps completed, the Honeypot is ready to attract potential SSH attacks.

## Step 3: Analyzing Attack Logs

After the Honeypot has run for a period, logs can be analyzed to gain insights into attacker behavior. Locate logs at `./cowrie/log/tty/cowrie.json` to review attempted commands by potential attackers.

By following these steps, this project provides a deeper understanding of SSH attack patterns, contributing to enhanced security measures.
