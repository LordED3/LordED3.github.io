---
layout: post
title: Building a DFIR Lab
date: 2024-04-08 01:10:00 -0500
categories: [School Project]
tags: [Cybersecurity, Computer Forensics]
image:
  path: assets/images/posts/2024-04-08-Building-a-DFIR-Lab/DFIR thumbnail.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---
## Building a DFIR Lab: A Hands-On Approach to Cybersecurity
As the world of cybersecurity evolves, so do the tools and environments required for effective digital forensics and incident response (DFIR). The ability to investigate cyber incidents and analyze malicious activity is paramount to defending against today's increasingly sophisticated threats. In this blog post, I'll walk you through the process of building a DFIR lab from scratch, combining both Windows and Linux environments to set up powerful, specialized tools for malware analysis, endpoint detection, and security monitoring.

## Objectives

1. **Build FlareVM**

   - Create a dedicated Windows 10 VM in VMware Workstation.
   - Prepare the VM to meet the prerequisites to install Flare VM.
   - Download FlareVM from the official GitHub repository.

   **Aim:** Establish a specialized Windows environment optimized for malware analysis and reverse engineering.

2. **Download and Setup REMnux VM on VMware Workstation**

   **Aim:** Setup a Linux-based platform equipped with a comprehensive suite of malware analysis tools and utilities.

3. **Install Wazuh as an EDR, XDR, and SIEM Solution**

   - Set up a dedicated virtual machine for Wazuh using Ubuntu or CentOS.
   - Install and configure the Wazuh manager, agents, and Elasticsearch as per the official documentation.

   **Aim:** Establish a robust security monitoring system capable of endpoint detection and response (EDR), extended detection and response (XDR), and security information and event management (SIEM).


## Building FlareVM

I successfully built FlareVM using the following steps:

1. **Creating the Base Windows 10 VM:** 
   I initiated a Windows 10 VM via VMware Workstation.

![VMware](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-02 173524.png){: w="900" h="900" }
_Windows 10 VMWare workstation Settings_

2. **Setting Flare VM Prerequisites:**
   After VM creation, I configured the prerequisites for Flare VM, including:
   - Disabling Windows automatic updates
   - Disabling Tamper Protection
   - Disabling Windows Defender
   These instructions can be found on Flare VM's [GitHub page.](https://github.com/mandiant/flare-vm)

3. **Activating Flare VM Script:**
   Once prerequisites were set, I activated the Flare VM script, which initiated the installation process. This process typically takes some time.

![VMware](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-02 200312.png){: w="900" h="900" }
_FlareVM Activation Script_

4. **Completing Flare VM Installation:**
   After a while, Flare VM installation was successfully completed. The entire process took approximately 1 hour 30 minutes to 2 hours.

![VMware](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-02 230923.png){: w="900" h="900" }
_Post Installation_

5. **Adding Additional Software:**
   I noticed that the Flare VM script did not install Autopsy and FTK Imager, so I manually added them.

![FLareVM](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-03 144947.png){: w="900" h="900" }
_Completed State_


## Downloading REMnux

The REMnux Virtual Machine was downloaded from [this link](https://docs.remnux.org/install-distro/get-virtual-appliance) and imported into VMware Workstation.

![REMnux](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-03 145201.png){: w="900" h="900" }
_Completed State_


## WAZUH Setup

WAZUH is a powerful open-source SIEM, EDR, XDR security solution. Here's how I set it up:

1. **Creating a Linux VM:** 
   I created a Linux VM, preferably Ubuntu, to serve as the WAZUH server.

![REMnux](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-03 154501.png){: w="900" h="900" }
_Ubuntu Wazuh_

2. **Installing Wazuh Server:** 
   Following the documentation, I installed the Wazuh Server. Once installed, we could manage it through the web interface and deploy agents on the endpoints.

![REMnux](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-03 160108.png){: w="900" h="900" }
_WAZUH installation_

3. **Deploying WAZUH Agents:**
   WAZUH agents were deployed to REMnux and FlareVM for comprehensive security monitoring.

![REMnux](assets/images/posts/2024-04-08-Building-a-DFIR-Lab/Blog PIc/Screenshot 2024-04-03 162907.png){: w="900" h="900" }
_Agents Deployed on Endpoints_


## Conclusion

Building a DFIR lab demonstrates my commitment to lifelong learning and professional development in cybersecurity. By creating a dedicated space for practical experimentation and exploration, I am laying the groundwork for my continuous growth and mastery in the field of digital forensics and incident response.
