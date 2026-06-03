# Phase 2 — Microsoft Defender for Endpoint, Sentinel & Enterprise Identity

![Platform](https://img.shields.io/badge/Platform-Proxmox_VE-orange) ![SIEM](https://img.shields.io/badge/SIEM-Elastic_8.19_%7C_Microsoft_Sentinel-blue) ![EDR](https://img.shields.io/badge/EDR-Microsoft_Defender_for_Endpoint-0078D4) ![MDM](https://img.shields.io/badge/MDM-Microsoft_Intune-0078D4) ![Identity](https://img.shields.io/badge/Identity-Microsoft_Entra_ID-0078D4) ![MITRE ATT&CK](https://img.shields.io/badge/MITRE_ATT%26CK-T1110.003-red) ![Status](https://img.shields.io/badge/Status-Active-brightgreen)

A continuation of the SOC Home Lab project. Phase 2 extends the existing Elastic Stack environment by integrating the full Microsoft enterprise security stack — Microsoft Defender for Endpoint, Microsoft Sentinel, Microsoft Intune, and Microsoft Entra Hybrid Join. Both Windows 11 endpoints are now managed through a unified cloud identity and security platform running in parallel with Elastic Stack.

**Key outcome:** The same Kali Linux password spray attack (T1110.003) is detected simultaneously in both Elastic Stack and Microsoft Sentinel, enabling a direct dual-SIEM comparison.

### Lab Summary

| Metric | Value |
|---|---|
| Endpoints | 2 |
| Servers | 2 |
| SIEMs | 2 (Elastic Stack + Microsoft Sentinel) |
| Identity Sources | Active Directory + Microsoft Entra ID |
| EDR | Microsoft Defender for Endpoint |
| MDM | Microsoft Intune |
| Attack Simulations | 1 |
| Custom Detection Rules | 1 (Sentinel KQL Analytics Rule) |

---

## Lab Architecture

![SOC Home Lab Architecture Diagram Phase 2](../diagrams/SOC%20Home%20Lab%20Phase%202%20Architecture%20Diagram.png)

### Architecture Highlights

- Hybrid identity using Active Directory and Microsoft Entra ID
- Dual SIEM deployment with Elastic Stack and Microsoft Sentinel running in parallel
- Microsoft Defender for Endpoint telemetry ingested through Microsoft Sentinel's Defender XDR connector
- Both Windows 11 endpoints enrolled and managed by Microsoft Intune
- Tailscale mesh VPN provides secure remote administration across all lab nodes
- Attack simulation performed from a dedicated Kali Linux VM on the same subnet

### What Changed from Phase 1

Phase 2 introduced the Microsoft cloud layer on top of the existing on-premises infrastructure. The core Proxmox environment, Active Directory domain, Elastic Stack SIEM, and Kali attacker remain unchanged. The following components were added:

- Microsoft Entra Connect Sync installed on the DC to bridge `lab.local` with `MensahCyberLab.onmicrosoft.com`
- Microsoft Entra Hybrid Join configured so both Win11 VMs are registered in both on-premises AD and Microsoft Entra ID simultaneously
- Microsoft Intune MDM enrollment for both endpoints
- Microsoft Defender for Endpoint onboarded on both endpoints via local script
- Microsoft Sentinel workspace connected to MDE via the Defender XDR data connector
- Tailscale expanded to include both Win11 VMs for full remote access across all lab nodes

### Why Victim1's IP Changed

Victim1's IP address changed from `192.168.1.238` (Phase 1) to `192.168.1.251` (Phase 2) because the VM was fully rebuilt from scratch. The original victim1 had a corrupted domain join state that could not be repaired in place. After exhausting all repair options, the VM was destroyed in Proxmox and rebuilt clean. The rebuilt VM received a new DHCP lease from the home router, resulting in the new IP. Full details are in the Problems Solved section below.

---

## Infrastructure

| VM | Role | OS | IP |
|---|---|---|---|
| Windows Server 2022 | Active Directory Domain Controller | Windows Server 2022 | 192.168.1.200 |
| Windows 11 Victim 1 | Domain Endpoint | Windows 11 Pro 25H2 | 192.168.1.251 |
| Windows 11 Victim 2 | Domain Endpoint | Windows 11 Pro 25H2 | 192.168.1.231 |
| Kali Linux | Attacker Machine | Kali Linux 2026.1 | 192.168.1.227 |
| Ubuntu Server | Elastic SIEM | Ubuntu 26.04 LTS | 192.168.1.232 |

Both Windows 11 endpoints are joined to the `lab.local` Active Directory domain and registered as Microsoft Entra Hybrid Joined devices.

---

## Remote Access — Tailscale

Tailscale was expanded in Phase 2 to include both Win11 VMs, enabling full remote SSH access and management from any network without port forwarding.

| Device | Tailscale IP |
|---|---|
| Proxmox Host | 100.91.205.44 |
| Elastic SIEM / Kibana | 100.65.16.119 |
| Victim 1 | 100.120.38.56 |
| Victim 2 | 100.92.222.61 |

---

## Microsoft Security Stack

### Microsoft Entra ID Tenant & Licensing
The lab tenant `MensahCyberLab.onmicrosoft.com` was provisioned using a Microsoft 365 E5 trial, which includes MDE P2, Microsoft Sentinel, Entra ID P2, and Intune at no cost.

### Microsoft Entra Connect Sync
Microsoft Entra Connect Sync was deployed on the Windows Server 2022 DC to synchronize the on-premises `lab.local` domain with the cloud tenant. The UPN suffix was configured as `@MensahCyberLab.onmicrosoft.com` so that user accounts are routable in Entra ID. Delta sync runs every 30 minutes.

### Microsoft Entra Hybrid Join
Both Win11 VMs are joined to `lab.local` on-premises and simultaneously registered in Microsoft Entra ID. The Service Connection Point (SCP) was written to AD via Entra Connect to enable automatic device registration. Hybrid Join status was verified on each VM using `dsregcmd /status`, confirming `AzureAdJoined: YES` and `DomainJoined: YES`.

![Entra ID Enrolled Devices](screenshots/Entra%20ID%20Enrolled%20Devices.png)

### Microsoft Intune MDM
MDM user scope was set to All in the Intune automatic enrollment settings. Both endpoints were enrolled using `deviceenroller.exe /AutoEnrollMDM` running as SYSTEM. Both devices show Compliant status in the Intune admin center with `labuser@MensahCyberLab.onmicrosoft.com` as the primary user.

![Intune Enrolled Devices](screenshots/Intune%20Enrolled%20Devices.png)

### Microsoft Defender for Endpoint
MDE is a cloud service with no on-premises component. Both endpoints were onboarded using a local script with Streamlined connectivity. The MDE sensor (`Sense` service) was confirmed running on both endpoints, and `Get-MpComputerStatus` verified that `AMServiceEnabled`, `BehaviorMonitorEnabled`, and signature currency were all healthy.

![MDE Device Inventory](screenshots/MDE%20Device%20Inventory.png)

---

## Microsoft Sentinel

### Workspace
The `mensahcyberlab-sentinel` workspace was created in East US and connected to the Microsoft Defender XDR unified portal. Resource group: `mensahcyberlab-rg`.

### Data Connector — Microsoft Defender XDR (10/10 tables connected)

| Table | Description |
|---|---|
| DeviceInfo | Machine and OS information |
| DeviceNetworkInfo | Network properties |
| DeviceProcessEvents | Process creation events |
| DeviceNetworkEvents | Network connections |
| DeviceFileEvents | File system events |
| DeviceRegistryEvents | Registry modifications |
| DeviceLogonEvents | Authentication events |
| DeviceImageLoadEvents | DLL loading events |
| DeviceEvents | Additional event types |
| DeviceFileCertificateInfo | Certificate information |

AlertInfo and AlertEvidence were also connected for MDE alert ingestion.

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Credential Access | Password Spraying | T1110.003 |
| Discovery | Account Discovery | T1087 |
| Initial Access | Valid Accounts (Potential Outcome) | T1078 |

---

## Detection Engineering

### Analytics Rule — Kali Password Spray Attack - MDE

**MITRE ATT&CK:** T1110.003 — Password Spraying  
**Tactic:** Credential Access  
**Severity:** Medium  
**Schedule:** Every 5 minutes, 1-hour lookback

```kusto
DeviceLogonEvents
| where TimeGenerated > ago(1h)
| where ActionType == "LogonFailed"
| where LogonType == "Network"
| summarize FailedAttempts = count(), TargetAccounts = dcount(AccountName)
    by RemoteIP, DeviceName, bin(TimeGenerated, 5m)
| where FailedAttempts > 5 and TargetAccounts > 1
| project TimeGenerated, RemoteIP, DeviceName, FailedAttempts, TargetAccounts
```

### Detection Logic Rationale

- `FailedAttempts > 5` reduces noise from isolated authentication failures and targets sustained attack behavior.
- `TargetAccounts > 1` differentiates password spraying from a single-user brute force attack — the defining characteristic of a spray is that one password is tried across many accounts.
- The 5-minute aggregation window balances detection sensitivity against false positives, catching rapid spray campaigns while avoiding alert fatigue from slow credential stuffing.

---

## Detection Flow

1. Kali Linux launches an SMB password spray attack against both Windows 11 endpoints.
2. The Windows endpoints generate failed authentication events captured by the MDE sensor.
3. Microsoft Defender for Endpoint collects the endpoint telemetry in real time.
4. Microsoft Defender XDR makes endpoint telemetry available to Microsoft Sentinel through the Defender XDR data connector.
5. The Sentinel analytics rule identifies the password spraying behavior using KQL.
6. An incident is automatically created and enriched with host and IP entities.
7. Elastic Stack independently detects the same activity using Sysmon and Elastic Agent logs.

---

## Attack Simulation

**Tool:** CrackMapExec  
**Protocol:** SMB (port 445)  
**Attacker:** `192.168.1.227` (Kali)  
**Targets:** `victim1.lab.local` (192.168.1.251) and `victim2.lab.local` (192.168.1.231)

```bash
crackmapexec smb 192.168.1.251 192.168.1.231 \
  -u administrator labuser guest \
  -p Password123! password123 Welcome1 Summer2026! Winter2026! Passw0rd! \
  --smb-timeout 30
```

The attack returned `STATUS_LOGON_FAILURE` across all credential combinations against both targets.

![Kali Password Spray Attack](screenshots/Kali%20Password%20Spray%20Attack%20on%20Both%20Victims.png)

### Advanced Hunting Query

```kusto
DeviceLogonEvents
| where Timestamp > ago(1h)
| where DeviceName in ("victim1.lab.local", "victim2.lab.local")
| where ActionType == "LogonFailed"
| where RemoteIP == "192.168.1.227"
| summarize FailedAttempts = count(), TargetAccounts = dcount(AccountName)
    by DeviceName, RemoteIP, bin(Timestamp, 5m)
| order by Timestamp desc
```

**Output:**
- `victim1.lab.local` — 18 failed attempts, 3 accounts targeted
- `victim2.lab.local` — 14 failed attempts, 3 accounts targeted

![Advanced Hunting Results](screenshots/Advanced%20Hunting%20in%20MDE.png)

### Sentinel Incident

| Field | Value |
|---|---|
| Incident ID | 1 |
| Name | Kali Password Spray Attack - MDE |
| Severity | Medium |
| Total Alerts | 34 |
| Category | Credential Access |
| Attacker IP | 192.168.1.227 (Kali) |
| Detection Latency | ~9 minutes |

![Microsoft Sentinel Logs](screenshots/Microsoft%20Sentinel%20Logs%20.png)

![Microsoft Defender Detections](screenshots/Microsoft%20Defender%20Detections.png)

---

## Incident Response Workflow

After the Sentinel incident was generated, the following analyst workflow was applied:

1. **Alert generated.** Sentinel created incident ID 1 — Kali Password Spray Attack - MDE — with Medium severity and Credential Access categorization.
2. **Analyst triage.** Reviewed the alert details, confirmed 34 alerts across multiple 5-minute detection windows, and identified two affected endpoints.
3. **Validate source IP.** Confirmed the source IP `192.168.1.227` maps to the Kali Linux attacker VM on the lab network.
4. **Review affected accounts.** Identified three targeted accounts across both endpoints — `administrator`, `labuser`, and `guest`.
5. **Determine attack scope.** Advanced Hunting confirmed the attack was limited to SMB authentication attempts with no successful logons — no lateral movement or follow-on activity observed.
6. **Containment decision.** In a production environment, the next steps would be to isolate the source IP via firewall policy and disable any accounts showing signs of compromise. In the lab context, the Kali VM was powered down.
7. **Documentation.** Incident findings, affected assets, attacker TTPs, and response actions were documented for the post-incident report.

---

## Evidence

| Screenshot | Description |
|---|---|
| `Kali Password Spray Attack on Both Victims.png` | CrackMapExec output showing failed SMB authentication attempts against both endpoints |
| `Advanced Hunting in MDE.png` | Advanced Hunting query results showing failed logon activity from the Kali attacker |
| `Microsoft Defender Detections.png` | Password spray detections generated by Microsoft Defender |
| `Microsoft Sentinel Logs.png` | Sentinel analytics rule output and log evidence |
| `MDE Device Inventory.png` | Defender for Endpoint inventory showing onboarded devices |
| `Entra ID Enrolled Devices.png` | Microsoft Entra Hybrid Joined devices |
| `Intune Enrolled Devices.png` | Intune-managed compliant endpoints |

---

## Detection Platform Comparison

| Feature | Elastic Stack | Microsoft Sentinel |
|---|---|---|
| Data source | Sysmon + Elastic Agent | Microsoft Defender XDR / MDE telemetry |
| Query language | KQL (Kibana) | Kusto (KQL) |
| Detection rule | Custom threshold rule | Scheduled analytics rule |
| Alert latency | ~2 minutes | ~5–10 minutes |
| Entity enrichment | Manual | Automatic (IP, Host mapping) |
| Incident management | Manual case creation | Automatic incident grouping |
| MITRE mapping | Manual tagging | Native ATT&CK integration |
| Cloud-native | No | Yes |

---

## Skills Demonstrated

- Microsoft Entra ID tenant provisioning and Microsoft 365 E5 licensing
- Microsoft Entra Connect Sync deployment and AD-to-cloud identity bridging
- Microsoft Entra Hybrid Join configuration via SCP and Group Policy
- Microsoft Intune MDM enrollment and compliance policy
- Microsoft Defender for Endpoint onboarding via local script with Streamlined connectivity
- Microsoft Sentinel workspace creation and data connector configuration
- Kusto (KQL) detection rule engineering in Microsoft Sentinel
- Dual SIEM architecture with parallel detection across Elastic Stack and Microsoft Sentinel
- MITRE ATT&CK T1110.003 detection and incident response
- CrackMapExec SMB password spray attack simulation
- VM rebuild and enterprise endpoint reconfiguration from scratch
- Tailscale mesh VPN expansion for full remote lab access

---

## Problems Solved

<details>
<summary><b>1. M365 Developer Sandbox Not Available</b></summary>

The Microsoft 365 Developer Program sandbox was unavailable on the new lab account because Microsoft tightened eligibility in 2024, now requiring a paid Visual Studio subscription. The issue was resolved by signing up for a Microsoft 365 Business Basic 30-day trial to bootstrap an organizational tenant, then adding the M365 E5 trial on top through the admin center billing catalog at no cost.

</details>

<details>
<summary><b>2. Intune Enrollment Blocked — Error -895156188</b></summary>

Every Intune enrollment attempt returned "Error response came from MDM terms of use page." The error persisted across Company Portal, Access Work or School, and direct enrollment methods. The issue was resolved by running `deviceenroller.exe /AutoEnrollMDM` as a scheduled task under `NT AUTHORITY\SYSTEM`, which bypassed the Terms of Use rendering issue entirely.

</details>

<details>
<summary><b>3. Victim1 — Broken Secure Channel (ERROR_NO_TRUST_SAM_ACCOUNT)</b></summary>

`nltest /sc_verify:lab.local` returned `ERROR_NO_TRUST_SAM_ACCOUNT`. Group Policy would not apply and `AzureAdJoined` remained NO. The root cause was that victim1 had never had a proper computer account created in Active Directory — it existed in a corrupted domain-joined state with no valid secure channel. Multiple repair attempts failed including `Test-ComputerSecureChannel -Repair`, `Reset-ComputerMachinePassword`, and direct Netlogon registry edits. NTLM being disabled on the DC blocked all credential-based unjoin attempts.

</details>

<details>
<summary><b>4. Victim1 Full VM Rebuild (IP Change: .238 → .251)</b></summary>

After exhausting all in-place repair options, VM 101 was destroyed in Proxmox and rebuilt from scratch using the Windows 11 25H2 ISO with VirtIO drivers. The rebuild process included a fresh Windows install, VirtIO guest tools, domain join to `lab.local`, Sysmon installation, Elastic Agent enrollment, MDE onboarding, Microsoft Entra Hybrid Join, and Intune enrollment — all completed successfully in the correct order.

Victim1's IP changed from `192.168.1.238` (Phase 1) to `192.168.1.251` (Phase 2) because the rebuilt VM received a new DHCP lease from the home router.

</details>

<details>
<summary><b>5. Stale Entra Device Object Blocking Hybrid Join</b></summary>

After the rebuild, `dsregcmd /join` failed with `error_missing_device`, referencing a device ID from the destroyed VM that was still cached locally in the registry. The issue was resolved by locating and deleting the stale key at `HKLM\SYSTEM\CurrentControlSet\Control\CloudDomainJoin\Diagnostics` and rebooting. `AzureAdJoined: YES` was confirmed on the next join attempt.

</details>

<details>
<summary><b>6. MDE Sense Service Failing to Start After Registry Changes</b></summary>

Registry changes made during SMB troubleshooting cleared the MDE onboarding configuration. The Sense service failed with "not onboarded and no onboarding parameter." The issue was resolved by re-downloading the local onboarding script from the Defender portal and re-running it as Administrator on victim1.

</details>

<details>
<summary><b>7. CrackMapExec NETBIOS Timeouts on Victim1</b></summary>

Port 445 was confirmed open via `nc -zv` but CrackMapExec returned NETBIOS connection timeouts. The issue was resolved by adding `--smb-timeout 30` to extend the timeout threshold for the SMB authentication handshake.

</details>

<details>
<summary><b>8. DeviceLogonEvents Not Populating After Re-onboard</b></summary>

After MDE re-onboarding, `DeviceProcessEvents` and `DeviceNetworkEvents` were flowing normally but `DeviceLogonEvents` returned no results. The root cause was that the MDE sensor initializes different telemetry categories at different intervals after onboarding, and authentication monitoring takes 15 to 30 minutes to fully initialize on a fresh onboard. The issue resolved itself after waiting for full initialization and re-running the attack.

</details>

---

## Lessons Learned

**Hybrid identity is foundational.** Before any Microsoft security tooling could be deployed, the identity layer had to be right. Microsoft Entra Connect Sync, Hybrid Join, and proper UPN configuration were prerequisites for everything else. MDE onboarding, Intune enrollment, and Sentinel data ingestion all depend on a healthy identity plane.

**Cloud SIEMs consume, they do not collect.** Microsoft Sentinel did not directly collect endpoint telemetry in this deployment. Instead, it consumed telemetry already collected by Microsoft Defender for Endpoint and exposed through the Defender XDR connector. Understanding this distinction changes how you think about data pipelines and coverage gaps.

**Dual SIEM operations surface platform tradeoffs.** Running Elastic Stack and Microsoft Sentinel in parallel on the same attack exposed meaningful differences. Elastic detected the attack in approximately 2 minutes using Sysmon process telemetry, while Sentinel detected it in approximately 9 minutes using MDE authentication telemetry. Neither is strictly better — they capture different signals, and a mature SOC uses both layers.

**Troubleshooting broken identity state is a real SOC skill.** The victim1 rebuild was caused by a corrupted domain join, a condition that appears frequently in enterprise environments when machines are imaged, cloned, or migrated without proper cleanup. Diagnosing `ERROR_NO_TRUST_SAM_ACCOUNT`, understanding secure channel failure, and knowing when to rebuild versus repair are skills that translate directly to production incident response.

**Telemetry initialization is not instantaneous.** MDE sensors initialize different telemetry categories at different intervals after onboarding. `DeviceLogonEvents` took 15 to 30 minutes to populate after a fresh onboard while `DeviceProcessEvents` and `DeviceNetworkEvents` were available almost immediately. In a real SOC, newly onboarded endpoints have a brief visibility gap that must be understood and accounted for.

---

## Repository Structure

```
phase-2-mde-sentinel/
├── README.md
└── screenshots/
    ├── Advanced Hunting in MDE.png
    ├── Entra ID Enrolled Devices.png
    ├── Intune Enrolled Devices.png
    ├── Kali Password Spray Attack on Both Victims.png
    ├── MDE Device Inventory.png
    ├── Microsoft Defender Detections.png
    ├── Microsoft Sentinel Logs.png
    └── SOC_Home_Lab_Architecture_Diagram_2.0.png
```

---

## Project Status

Phase 2 is complete and operational.

The lab now supports:
- Active Directory and Microsoft Entra Hybrid Identity
- Microsoft Intune device management
- Microsoft Defender for Endpoint (EDR)
- Microsoft Sentinel cloud SIEM
- Elastic Stack self-hosted SIEM
- Parallel attack detection across two independent security platforms

Future phases will focus on advanced attack simulations, detection engineering, automation, threat hunting, and SOAR workflows. As the lab continues to evolve, new attack simulations, detections, and infrastructure components will be documented in this [SOC Home Lab Repository](https://github.com/KingMike77/SOC-Home-Lab).
