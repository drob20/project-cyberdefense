# Case 01 — External Scan Against 10.0.1.6 (SMB)

This directory documents a beginner-friendly network scanning case performed in a home SOC lab. The goal is to demonstrate how a basic Nmap scan appears on the network and how Suricata and Wazuh record the activity.

This case is intentionally simple and will be expanded later with multiple attack vectors, full triage steps, and response workflow examples.

---

## ✅ Lab Overview

| Component             | IP / VLAN          | Role                                          |
| --------------------- | ------------------ | --------------------------------------------- |
| pfSense + Suricata    | 10.0.2.1 (VLAN 20) | Firewall & Intrusion Detection (logs → Wazuh) |
| Wazuh Manager         | 10.0.2.x (VLAN 20) | SIEM / Log aggregation                        |
| Kali Linux (Attacker) | 10.0.3.2 (VLAN 30) | Runs Nmap scans                               |
| Windows Host (Target) | 10.0.1.6 (VLAN 10) | SMB service on port 445                       |

**Network segmentation:**

```
VLAN 10 -> Windows Endpoints
VLAN 20 -> Wazuh / Logging / Infrastructure
VLAN 30 -> Attacker / Testing Network
```

---

## 🎯 Scan Objective

Perform a controlled external scan from the attacker VLAN to the Windows target and observe:

* Network behavior
* Suricata alert generation
* Wazuh log visibility

---

## 🛠 Scan Command

Run from Kali Linux:

Also ran on Zenmap to see Topology for visual purposes
nmap -T4 -A -v -Pn 10.0.1.6
```

This is identical to Zenmap preset **Intense Scan, No Ping**.

<img width="900" height="384" alt="znmap" src="https://github.com/user-attachments/assets/0cc0ed0a-3e47-4746-af3c-8affb567495c" />

## 🧾 Scan Results (Summary)

| Detail                 | Result                          |
| ---------------------- | ------------------------------- |
| Host Discovery         | Responded (Ping disabled)       |
| Open Port              | **445/tcp (SMB)**               |
| Service Fingerprinting | SMB negotiation attempted       |
| Alert Triggered        | Suricata SMB dialect inspection |


---

## 📡 Suricata Evidence (from pfSense → Wazuh)

| Field            | Value                                     |
| ---------------- | ----------------------------------------- |
| Source IP        | `10.0.3.2` (Kali Attacker)                |
| Destination IP   | `10.0.1.6` (Windows Target)               |
| Destination Port | `445/tcp`                                 |
| Suricata Alert   | `SURICATA SMB malformed request dialects` |
| Severity         | Medium (3)                                |
| Action Taken     | Allowed (observed, not blocked)           |

This confirms the scan reached SMB protocol negotiation, allowing the IDS to analyze the session.

<img width="1822" height="890" alt="log1" src="https://github.com/user-attachments/assets/bb44f37b-cc37-4e30-b393-a0e4074491d2" />
<img width="1804" height="878" alt="log2" src="https://github.com/user-attachments/assets/3f78ab11-d139-4db1-9ce7-b5e8a18ea16a" />
<img width="1838" height="776" alt="log3" src="https://github.com/user-attachments/assets/a0da07ad-9132-4e64-8708-d92a1e347b77" />

---

## 🔥 Windows Firewall Note (Important for Reproducibility)

By default, Windows blocks inbound SMB traffic.

To allow the scan and generate meaningful alerts, **a temporary firewall rule was created**:

```
Allow inbound ANY TCP during test only.
```
<img width="1946" height="1156" alt="firewall rule to enable connection " src="https://github.com/user-attachments/assets/a3bfb7da-7e41-4b80-95b8-19b35980a74a" />

After capturing evidence, the rule was **removed to restore a secure posture**.

This demonstrates awareness of:

* default host-layer protections
* controlled testing methodology
* restoring baseline security after testing

---

## 🧠 Key Takeaways (Beginner Learning)

* Network segmentation determines visibility and access
* A scan must *reach* the service for Suricata to see protocol artifacts
* Wazuh correctly ingests Suricata EVE alerts for analyst review
* Even a basic scan can produce useful detection evidence

---

## 🔄 Future Expansion Roadmap

| Planned Enhancement                    | Outcome                                |
| -------------------------------------- | -------------------------------------- |
| Add case triage worksheet              | Demonstrate SOC investigation workflow |
| Add Suricata + Wazuh correlation rules | Reduce noise & improve alert fidelity  |

---

## ✔ Status

This case is **complete for beginner SOC demonstration**.
Rules, PCAP analysis, and host telemetry correlation will be added in the next phase.
