# project-cyberdefense
A Proxmox-based Security Operations Center (SOC) lab environment simulating enterprise blue team operations using pfSense, Wazuh, and Windows/Linux endpoints.
This project showcases a fully virtualized Security Operations Center (SOC) lab built on Proxmox VE. The environment simulates a small enterprise network segmented into multiple VLANs with pfSense for routing and firewalling, Wazuh for SIEM and endpoint monitoring, Kali Linux to simulate attacker activity, and a mix of Windows Server, Windows 11, and Ubuntu systems.

## Overview
The purpose of this lab is to replicate real-world SOC workflows—log collection, alert triage, threat detection, and response—within a controlled, virtualized environment. It serves as a platform for continuous hands-on learning in cybersecurity, focusing on blue team operations, defensive monitoring, and adversary simulation.

### Key Components
Proxmox VE — virtualization and resource management

pfSense — firewall, routing, VLAN segmentation

Wazuh — SIEM / EDR / centralized log analysis

Kali Linux — attacker VM for red-team simulation and traffic generation

Windows Server & Clients — Active Directory, domain-joined endpoints for detection scenarios

Ubuntu — analysis/administrative tools (forensics, tooling, scripts)

This lab demonstrates the design, deployment, and maintenance of a defensive cyber infrastructure that mirrors small-to-medium enterprise operations, providing practical experience in threat visibility, incident response, detection engineering, and adversary emulation.
