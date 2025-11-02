# project-cyberdefense
A Proxmox-based Security Operations Center (SOC) lab environment simulating enterprise blue team operations using pfSense, Wazuh, and Windows/Linux endpoints.
This project showcases a fully virtualized Security Operations Center (SOC) lab built on Proxmox VE. The environment simulates a small enterprise network segmented into multiple VLANs with pfSense for routing and firewalling, Wazuh for SIEM and endpoint monitoring, Kali Linux to simulate attacker activity, and a mix of Windows Server, Windows 11, and Ubuntu systems.

## Overview
The purpose of this lab is to replicate real-world SOC workflows—log collection, alert triage, threat detection, and response—within a controlled, virtualized environment. It serves as a platform for continuous hands-on learning in cybersecurity, focusing on blue team operations, defensive monitoring, and adversary simulation.

## Key Components
Proxmox VE — virtualization and resource management

pfSense — firewall, routing, VLAN segmentation

Wazuh — SIEM / EDR / centralized log analysis

Kali Linux — attacker VM for red-team simulation and traffic generation

Windows Server & Clients — Active Directory, domain-joined endpoints for detection scenarios

Ubuntu — analysis/administrative tools (forensics, tooling, scripts)

This lab demonstrates the design, deployment, and maintenance of a defensive cyber infrastructure that mirrors small-to-medium enterprise operations, providing practical experience in threat visibility, incident response, detection engineering, and adversary emulation.

## Network Architecture

| VLAN | Subnet | Purpose | Example Systems |
|------|---------|----------|----------------|
| — | Home LAN (unsegmented) | Management and Proxmox connectivity | Proxmox host, pfSense WAN interface |
| VLAN 10 | 10.0.1.0/24 | Windows Sever and user endpoint | Windows Server, Windows 11 clients |
| VLAN 20 | 10.0.2.0/24 | SOC infrastructure and monitoring | Wazuh server, Ubuntu admin box |
| VLAN 30 | 10.0.3.0/24 | Attacker network for adversary simulation | Kali Linux |


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/18d1296c-2574-4add-b611-ad55fd81fbbe" />




### Network Flow Summary 
- The **Proxmox host** itself is **not part of the VLANs** and resides on a separate management/home LAN for stability and out-of-band access.
- **pfSense** functions as the main firewall and router for VLANs 10, 20, and 30, enforcing inter-VLAN security policies.  
- **Proxmox** hosts all virtual machines and provides internal virtual bridges (`vmbr1`, `vmbr2`, etc.) connecting VLAN-tagged traffic to pfSense.  
- **Wazuh** collects logs and telemetry from all monitored systems through the Wazuh Agent.  
- **Kali Linux** is isolated on the attacker VLAN, generating simulated threat activity to test detection and response.  
- **Windows Server (Active Directory)** handles domain services (DNS, DHCP, authentication) for Windows endpoints on VLAN 10.  

### Diagram Placeholder

