# SOC Home Lab

![Platform](https://img.shields.io/badge/Platform-Proxmox-orange)
![SIEM](https://img.shields.io/badge/SIEM-Elastic%208.19-purple)
![OS](https://img.shields.io/badge/OS-Windows%20%7C%20Linux-blue)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1110.003-red)
![Status](https://img.shields.io/badge/Status-In%20Progress-success)

A home Security Operations Center (SOC) lab built from scratch — covering physical hardware setup, hypervisor configuration, virtual machine deployment, SIEM configuration, and attack simulation with detection engineering.

The lab simulates a real corporate environment where an attacker targets domain-joined Windows endpoints monitored by a centralized SIEM.

This is the first phase of an evolving project. This repository serves as the main hub for the SOC Home Lab — as the lab grows, new phases and attack simulations will be documented here.

---

## Lab Architecture

<p align="center">
  <img src="diagrams/SOC%20Home%20Lab%20Architecture%20Diagram.png" width="1100">
</p>

<p align="center">
Detection Engineering & Adversary Emulation Lab running on Proxmox VE hosted on an HP EliteDesk 800 G5 SFF.
</p>

---

## Table of Contents

- [Physical Infrastructure](#physical-infrastructure)
- [Hypervisor — Proxmox VE](#hypervisor--proxmox-ve)
- [Infrastructure](#infrastructure)
- [SIEM Stack — Elastic 8.19](#siem-stack--elastic-819)
- [Remote Access — Tailscale](#remote-access--tailscale)
- [Attack Simulation](#attack-simulation)
- [Detection Engineering](#detection-engineering)
- [Problems Solved](#problems-solved)
- [Skills Demonstrated](#skills-demonstrated)
- [Repository Structure](#repository-structure)

---

## Physical Infrastructure

### Primary Host — HP EliteDesk 800 G5 SFF

- Intel Core i5-9500 (6 cores)
- 24GB DDR4 RAM
- Samsung 860 EVO 500GB SSD
- Connected to AT&T Fiber home network

### Setup Process

Opened the machine, removed the original HDD, installed the SSD, and configured the BIOS to boot from the new drive before installing Proxmox VE.

---

## Hypervisor — Proxmox VE

Proxmox VE was installed directly on the SSD for fast VM performance.

### Configuration Steps

- Configured static IP (`192.168.1.101`) via `/etc/network/interfaces`
- Set up Linux bridge (`vmbr0`) so all VMs share the same subnet
- Added secondary storage pool (`vmdata`) for VM disk images
- Disabled enterprise repository and enabled free community repository
- Installed Tailscale for secure remote administration

---

## Infrastructure

| VM | Role | OS | IP |
|---|---|---|---|
| Windows Server 2022 | Active Directory Domain Controller | Windows Server 2022 | 192.168.1.200 |
| Windows 11 Victim 1 | Domain Endpoint | Windows 11 Pro | 192.168.1.238 |
| Windows 11 Victim 2 | Domain Endpoint | Windows 11 Pro | 192.168.1.231 |
| Kali Linux | Attacker Machine | Kali Linux 2026.1 | 192.168.1.227 |
| Ubuntu Server | Elastic SIEM | Ubuntu 26.04 LTS | 192.168.1.232 |

Both Windows 11 endpoints are joined to the `lab.local` Active Directory domain managed by the Windows Server 2022 domain controller.

---

## SIEM Stack — Elastic 8.19

Deployed on Ubuntu Server:

- **Elasticsearch** — log indexing and storage
- **Kibana** — dashboards, visualizations, and detections
- **Fleet Server** — centralized agent management

### Endpoint Telemetry

Elastic Agent is installed on all systems and enrolled into Fleet under a centralized lab policy.

The Windows endpoints additionally run:

- Sysmon (SwiftOnSecurity configuration)
- Windows Security logging
- PowerShell logging
- Process creation telemetry
- Network connection telemetry

---

## Remote Access — Tailscale

Tailscale was installed on both the Proxmox host and the Ubuntu SIEM server for secure remote access without port forwarding.

| Device | Tailscale IP |
|---|---|
| Proxmox Host | `100.91.205.44` |
| Elastic SIEM / Kibana | `100.65.16.119` |

This allows the environment to be monitored and administered securely from any network.

---

## Attack Simulation

### Password Spray Attack

**MITRE ATT&CK: T1110.003 — Password Spraying**

Simulated a password spray attack from Kali Linux against all Windows systems using SMB over TCP port 445.

### Commands Used

```bash
smbclient //192.168.1.200/C$ -U administrator%wrongpassword
smbclient //192.168.1.238/C$ -U administrator%wrongpassword
smbclient //192.168.1.231/C$ -U administrator%wrongpassword
```

### Evidence Captured

- Windows Security Event ID `4625`
- Failed authentication attempts
- Source IP: `192.168.1.227`
- Events collected centrally in Elastic SIEM

---

## Detection Engineering

Created a custom threshold-based detection rule in Elastic Security.

### Rule Configuration

| Setting | Value |
|---|---|
| Rule Type | Threshold |
| Index | `logs-*` |
| Severity | High |
| Risk Score | 73 |
| Group By | `host.name` |
| Threshold | 5 failed logons |

### KQL Query

```kql
event.code: "4625" and source.ip: "192.168.1.227"
```

### Result

High-severity alerts successfully triggered for:

- `win-c2kjctd2otq`
- `victim1`
- `victim2`

---

## Problems Solved

<details>
<summary><strong>Windows firewall blocking SMB traffic</strong></summary>

Windows Firewall automatically re-enabled after reboot, preventing SMB attack traffic.

### Fix

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

</details>

<details>
<summary><strong>Elastic Agent enrollment token corruption</strong></summary>

Enrollment tokens pasted through the Proxmox console corrupted special characters.

### Fix

Enabled SSH access on each VM and enrolled agents directly through SSH sessions.

</details>

<details>
<summary><strong>Elasticsearch memory pressure</strong></summary>

Elasticsearch began dropping events due to insufficient memory allocation.

### Fix

Increased Ubuntu SIEM VM memory allocation from 3GB to 8GB within Proxmox.

</details>

<details>
<summary><strong>2-hour log ingestion delay</strong></summary>

Logs appeared delayed inside Kibana due to timezone mismatch.

### Root Cause

Windows VMs were configured for Pacific Standard Time instead of Central Time.

### Fix

Corrected timezone settings and resynchronized Windows Time service.

</details>

<details>
<summary><strong>Detection rule not firing</strong></summary>

The threshold rule executed successfully but generated no alerts.

### Root Cause

Timestamp field mismatch and insufficient look-back window.

### Fix

- Set timestamp override to `@timestamp`
- Increased look-back window to 24 hours

</details>

---

## Skills Demonstrated

- Physical hardware upgrades and SSD installation
- Proxmox VE deployment and virtualization management
- Linux networking and bridge configuration
- Active Directory domain administration
- Windows endpoint management
- Elastic Stack deployment and administration
- Fleet-based centralized endpoint management
- Sysmon deployment and telemetry collection
- Detection engineering using KQL
- MITRE ATT&CK mapping
- Password spray attack simulation
- Threat hunting and log analysis in Kibana
- Remote infrastructure access using Tailscale

---

## Repository Structure
```
/
├── README.md
├── screenshots/
│   └── README.md
├── diagrams/
│   └── SOC Home Lab Architecture Diagram.png
├── configs/
│   ├── sysmon-config.xml
│   └── detection-rule.json
└── attack-scripts/
    └── password-spray.md
```
---

## Project Status

This is the first phase of the SOC Home Lab. This repository is the main hub for the project — as the lab continues to evolve, new attack simulations, detections, and infrastructure components will be documented here.
