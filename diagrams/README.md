# Diagrams

This folder contains architecture and network diagrams for the SOC Home Lab.

---

## SOC Home Lab Architecture Diagram

**File:** `SOC Home Lab Architecture Diagram 1.0.png`

![SOC Home Lab Architecture Diagram](SOC%20Home%20Lab%20Architecture%20Diagram%201.0.png)

This diagram shows the full architecture of the SOC Home Lab and how all components connect and interact with each other.

All 5 virtual machines run on a single Proxmox VE host — an HP EliteDesk 800 G5 SFF — and share the same internal network (`192.168.1.0/24`).

The Windows Server 2022 Domain Controller (`192.168.1.200`) manages the `lab.local` Active Directory domain. Both Windows 11 endpoints — Victim 1 (`192.168.1.238`) and Victim 2 (`192.168.1.231`) — are joined to that domain, simulating a real corporate environment with managed workstations.

Kali Linux (`192.168.1.227`) acts as the attacker machine on the same network segment, simulating an internal threat actor targeting the Windows machines over SMB port 445.

All 5 machines run Elastic Agent enrolled in Fleet, which is managed by the Fleet Server running on the Ubuntu SIEM (`192.168.1.232`). The three Windows machines additionally run Sysmon for deep endpoint telemetry. All logs and events are collected by Elasticsearch and visualized in Kibana, where custom detection rules fire alerts when attack patterns are identified.

Tailscale VPN is installed on both the Proxmox host (`100.91.205.44`) and the Ubuntu SIEM (`100.65.16.119`), allowing the entire lab to be accessed, monitored, and administered securely from any network without port forwarding.
