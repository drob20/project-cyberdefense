# Case 01 — External Scan Against 10.0.1.6 (SMB)

This project demonstrates an external network scan performed inside a segmented home SOC lab. The goal is to show how a basic Nmap scan appears on the network and how Suricata and Wazuh capture, log, and display the event. This is the first case and will be expanded with additional attack vectors, triage, and response workflows.

## Lab Overview

| Component             | IP / VLAN          | Role                                          |
|----------------------|--------------------|-----------------------------------------------|
| pfSense + Suricata   | 10.0.2.1 (VLAN 20) | Firewall & Intrusion Detection (logs → Wazuh) |
| Wazuh Manager        | 10.0.2.x (VLAN 20) | SIEM / Log aggregation                        |
| Kali Linux (Attacker)| 10.0.3.2 (VLAN 30) | Runs Nmap scans                               |
| Windows Host (Target)| 10.0.1.6 (VLAN 10) | SMB service running on port 445               |

```
VLAN 10 -> Windows Endpoints
VLAN 20 -> Wazuh / Logging / Infrastructure
VLAN 30 -> Attacker / Testing Network
```

## Scan Objective

Perform a controlled external scan from the attacker VLAN (10.0.3.2) to the Windows target (10.0.1.6) and observe:
- Network behavior
- Suricata inspection and alerting
- Wazuh visibility and event structure

## Scan Execution

Run from Kali Linux:
```bash
nmap -T4 -A -v -Pn 10.0.1.6
```

Zenmap topology visualization:
![Zenmap Topology](https://github.com/user-attachments/assets/0cc0ed0a-3e47-4746-af3c-8affb567495c)

## Scan Results (Summary)

| Detail                 | Result                          |
|------------------------|---------------------------------|
| Host Discovery         | Responded (Ping disabled)       |
| Open Port              | **445/tcp (SMB)**               |
| Service Fingerprinting | SMB negotiation attempted       |
| Alert Triggered        | Suricata SMB dialect inspection |

## Suricata Evidence (pfSense → Wazuh)

| Field            | Value                                     |
|------------------|-------------------------------------------|
| Source IP        | `10.0.3.2` (Attacker)                     |
| Destination IP   | `10.0.1.6` (Target)                       |
| Destination Port | `445/tcp`                                 |
| Suricata Alert   | `SURICATA SMB malformed request dialects` |
| Severity         | Medium (3)                                |
| Action Taken     | Allowed (Observed, Not Blocked)           |

![log1](https://github.com/user-attachments/assets/bb44f37b-cc37-4e30-b393-a0e4074491d2)
![log2](https://github.com/user-attachments/assets/3f78ab11-d139-4db1-9ce7-b5e8a18ea16a)
![log3](https://github.com/user-attachments/assets/a0da07ad-9132-4e64-8708-d92a1e347b77)

This confirms Suricata parsed the SMB negotiation attempt and forwarded the alert to Wazuh for analyst review.

## Windows Firewall Adjustment (Important for Reproducibility)

By default, Windows blocks inbound SMB. To allow the test to be visible to Suricata and Wazuh, a temporary firewall rule was enabled:

```
Allow inbound ANY TCP during test only.
```

![Windows Firewall Rule](https://github.com/user-attachments/assets/a3bfb7da-7e41-4b80-95b8-19b35980a74a)

After capturing evidence, the firewall rule was removed to restore secure posture.


## 💾 pfSense Log capture & PCAP Evidence 
<img width="1158" height="620" alt="loggsexmaple" src="https://github.com/user-attachments/assets/97d35f56-c9c0-48be-8c72-5be0b7ace50e" />

- I was able to see and log the live traffic from the scan.
  - Log into pfSense GUI (https://<pfsense_ip>/) with admin credentials.
  - Navigate to Diagnostics → Packet Capture.
  - Interface: choose the interface that sees the traffic (e.g., vnnet1, lan, or the mirror/tap interface).
  - Host Address: (optional) (source) or (dest) to filter.
  - Port: 445 (to capture SMB only) — or leave blank to capture full traffic (I used full TCP traffic).
  - Count / Length: optional (leave default to capture entire session).
  - Click Start. Re-run the Nmap scan from Kali while capture is running.
  - When the scan completes, click Stop. Then click Download Capture to save the .pcap file to your analysis machine. Save as smb_probe.pcap.
Inspect locally with Wireshark / tshark:

## Key Takeaways

- Network segmentation determines what traffic is visible and to whom
- For Suricata to analyze SMB, the SMB service must be reachable
- Wazuh provides structured alert data useful for SOC triage
- Even a basic scan provides meaningful detection practice


## Future Expansion Roadmap

| Planned Improvement                   | Purpose                                   |
|--------------------------------------|-------------------------------------------|
| Add PCAP analysis section            | Show packet-level SMB negotiation         |
| Add Sysmon to Windows host           | Correlate network → local process actions |
| Add incident triage worksheet        | Walk through analytical decision-making   |
| Add Suricata/Wazuh rule tuning       | Improve alert accuracy and context        |

## Status

This case is complete as a beginner SOC / detection engineering demonstration.  
Next phase will include:
- PCAP packet-by-packet analysis
- Sysmon host telemetry correlation
- Response playbook workflow
