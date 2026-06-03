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

---

## SOC Home Lab Phase 2 Architecture Diagram

**File:** `SOC Home Lab Phase 2 Architecture Diagram.png`

![SOC Home Lab Phase 2 Architecture Diagram](SOC%20Home%20Lab%20Phase%202%20Architecture%20Diagram.png)

Phase 2 extends the lab by integrating the full Microsoft enterprise security stack alongside the existing on-premises infrastructure. The same Proxmox host, Active Directory domain, Elastic Stack SIEM, and Kali attacker from Phase 1 remain unchanged.

**Microsoft cloud layer added in Phase 2:**

Microsoft Entra Connect Sync was deployed on the Windows Server 2022 DC to synchronize the `lab.local` on-premises domain with the cloud tenant `MensahCyberLab.onmicrosoft.com`. This bridges on-premises Active Directory with Microsoft Entra ID, enabling Hybrid Identity across the environment.

Both Windows 11 endpoints were configured for Microsoft Entra Hybrid Join, meaning they are simultaneously joined to the on-premises `lab.local` domain and registered in Microsoft Entra ID. This mirrors how enterprise endpoints are managed in a modern hybrid corporate environment.

Microsoft Intune MDM was enrolled on both endpoints, providing cloud-based device management and compliance monitoring. Both devices show Compliant status in the Intune admin center.

Microsoft Defender for Endpoint was onboarded on both Win11 endpoints via local script, providing EDR telemetry including process events, network connections, file activity, registry changes, and authentication events — all collected in real time and made available through the Microsoft Defender XDR platform.

Microsoft Sentinel was deployed as a second cloud-native SIEM. The Microsoft Defender XDR data connector was configured with all 10 MDE endpoint telemetry tables connected, feeding live endpoint data into Sentinel for detection, hunting, and incident management. A custom KQL analytics rule was written to detect password spray attacks — the same attack pattern detected by Elastic Stack in Phase 1 — enabling a direct dual-platform detection comparison.

**Changes from Phase 1:**

Victim1's IP changed from `192.168.1.238` (Phase 1) to `192.168.1.251` (Phase 2) due to a full VM rebuild required to resolve a corrupted domain join state where the secure channel between the endpoint and the DC had broken down. The rebuilt VM received a new DHCP lease from the home router. Full details are documented in the Phase 2 README.

Tailscale was expanded to include both Win11 VMs — Victim 1 (`100.120.38.56`) and Victim 2 (`100.92.222.61`) — enabling full remote SSH access and management of all lab nodes from any network without port forwarding.
