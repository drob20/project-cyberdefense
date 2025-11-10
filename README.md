# project-cyberdefense
A Proxmox-based Security Operations Center (SOC) lab environment simulating enterprise blue team operations using pfSense, Wazuh, and Windows/Linux endpoints.
This project showcases a fully virtualized Security Operations Center (SOC) lab built on Proxmox VE. The environment simulates a small enterprise network segmented into multiple VLANs with pfSense for routing and firewalling, Wazuh for SIEM and endpoint monitoring, Kali Linux to simulate attacker activity, and a mix of Windows Server, Windows 10, and Ubuntu systems.

## Overview
The purpose of this lab is to replicate real-world SOC workflows—log collection, alert triage, threat detection, and response—within a controlled, virtualized environment. It serves as a platform for continuous hands-on learning in cybersecurity, focusing on blue team operations, defensive monitoring, and adversary simulation.

## Key Components
Proxmox VE — virtualization and resource management

pfSense — firewall, routing, VLAN segmentation

Suricata — network IDS/IPS for deep packet inspection, rule-based threat detection, and alert forwarding to Wazuh

Wazuh — SIEM / EDR / centralized log analysis

Kali Linux — attacker VM for red-team simulation and traffic generation

Windows Server & Clients — Active Directory, domain-joined endpoints for detection scenarios

Ubuntu — analysis/administrative tools (forensics, tooling, scripts)

This lab demonstrates the design, deployment, and maintenance of a defensive cyber infrastructure that mirrors small-to-medium enterprise operations, providing practical experience in threat visibility, incident response, detection engineering, and adversary emulation.

## Network Architecture

| VLAN | Subnet | Purpose | Example Systems |
|------|---------|----------|----------------|
| — | Home LAN (unsegmented) | Management and Proxmox connectivity | Proxmox host, pfSense WAN interface |
| VLAN 10 | 10.0.1.0/24 | Windows Sever and user endpoint | Windows Server, Windows 10/11 clients |
| VLAN 20 | 10.0.2.0/24 | SOC infrastructure and monitoring | Wazuh server, Ubuntu admin box |
| VLAN 30 | 10.0.3.0/24 | Attacker network for adversary simulation | Kali Linux |


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/fb82a829-8d34-4b92-9a55-4dd8f3844332" />




### Network Flow Summary 
- The **Proxmox host** itself is **not part of the VLANs** and resides on a separate management/home LAN for stability and out-of-band access.
- **pfSense** functions as the main firewall and router for VLANs 10, 20, and 30, enforcing inter-VLAN security policies.
- **Suricata** (IDS/IPS) — deployed to inspect traffic and generate alerts; integrated with pfSense or run as a dedicated VM/container attached to a mirrored/SPAN or inline path. Alerts and EVE JSON logs forwarded to Wazuh for central correlation. 
- **Proxmox** hosts all virtual machines and provides internal virtual bridges (`vmbr1`, `vmbr2`, etc.) connecting VLAN-tagged traffic to pfSense.  
- **Wazuh** collects logs and telemetry from all monitored systems through the Wazuh Agent.  
- **Kali Linux** is isolated on the attacker VLAN (Can be moved to internal VLAN 20), generating simulated threat activity to test detection and response.  
- **Windows Server (Active Directory)** handles domain services (DNS, DHCP, authentication) for Windows endpoints on VLAN 10.

## Setup & Deployment Overview

This section outlines how the SOC lab was deployed and configured on Proxmox VE. Each stage focuses on creating a realistic, isolated environment for security monitoring and adversary simulation.

### 1. Proxmox Installation & Host Configuration
- Installed **Proxmox VE** on a dedicated **AMD-based host** with virtualization support (AMD-V enabled).   
- Configured local storage pools for VM images (NVMe SSD for high-speed operations and caching, a SATA SSD for VM disk images, HDD's for backups, ISO storage, and archived data).  
- Created multiple **virtual bridges** (`vmbr1`, `vmbr2`, `vmbr3`) to represent VLAN 10, 20, and 30.  
- The host itself resides on the **home LAN** (not part of VLANs) for management access.

### 2. pfSense Deployment
- Deployed **pfSense** as a virtual firewall and router on Proxmox, connected to VLANs 10, 20, and 30.  
- Configured **VLAN interfaces**:  
  - VLAN 10 – Windows domain network (handled by Windows Server for DHCP/DNS)  
  - VLAN 20 – SOC monitoring network (pfSense provides DHCP/DNS)  
  - VLAN 30 – Attacker network (pfSense provides DHCP/DNS)  
- Implemented **NAT and inter-VLAN routing** with strict firewall rules to isolate traffic between VLANs.  
- Forwarded **DNS queries from VLAN 10** to the Windows Server to maintain Active Directory name resolution consistency.  
- Configured **pfSense DNS Resolver** for VLANs 20 and 30 and set gateway access controls to limit exposure between zones.


### 3. Wazuh Server Setup
- Deployed an **Ubuntu LXC container** on VLAN 20 to host the **Wazuh Manager**, providing SIEM and endpoint monitoring functionality with minimal resource overhead.  
- Installed and configured **Wazuh Manager**, **Filebeat**, and the **Wazuh Dashboard** services within the container.  
- Used a separate **Ubuntu virtual machine** on the same VLAN as a **management workstation** to securely access the Wazuh web GUI and perform SOC analysis.  
- Connected Windows and Linux endpoints to the Wazuh Manager using the **Wazuh Agent** for centralized log collection and alerting.  
- Verified agent registration, confirmed active status, and generated test alerts to validate event visibility and rule accuracy.


### 4. Windows Server & Client Configuration
- Installed **Windows Server 2022** on VLAN10 and promoted it to a **Domain Controller**.  
- Configured **Active Directory**, **DNS**, and **DHCP** roles.  
- Joined **Windows 10 clients** to the domain.  
- Deployed Wazuh agents via manual installation.

### 5. Kali Linux (Attacker VM)
- Deployed a **Kali Linux VM** on VLAN30 for red-team simulation.  
- Verified isolation between attacker and production VLANs using pfSense firewall policies.  
- Used Kali for reconnaissance (Nmap), credential attacks, and exploit simulation to generate SOC alerts.

### 6. Validation & Testing
- Verified all systems had network connectivity to pfSense and Wazuh Manager.  
- Confirmed Wazuh received endpoint logs, pfSense firewall logs, and system events.  
- Executed basic attack simulations (brute-force, scans, privilege escalation) to test alerting accuracy.

### 7. Management & Maintenance
- Snapshotted each VM after successful configuration.  
- Backed up pfSense and Wazuh configurations for quick recovery.  
- Regularly update VMs and Wazuh components to maintain functionality and security.

---

This configuration provides a stable, realistic environment for practicing SOC workflows, incident response, and defensive analysis across diverse operating systems and network layers.



