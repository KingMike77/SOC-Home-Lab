# Screenshots

This folder contains screenshots documenting the SOC Home Lab — covering the infrastructure, agent management, attack simulation, log evidence, and detection alerts.

---

## Proxmox — All 5 VMs Running

![Proxmox all 5 VMs Running](Proxmox%20all%205%20VM's%20Running%20.png)

Shows all 5 virtual machines running simultaneously on the Proxmox VE hypervisor — Windows Server 2022, Windows 11 Victim 1, Windows 11 Victim 2, Kali Linux, and Ubuntu Elastic SIEM. Confirms the full lab infrastructure is operational.

---

## Fleet — All 5 Agents Healthy

![Fleet all 5 agents Healthy](Fleet-%20all%205%20agents%20Healthy.png)

Shows all 5 Elastic Agents enrolled in Fleet and reporting as Healthy — kali-attacker, Victim1, VICTIM2, WIN-C2KJCTD2OTQ, and elastic-siem. Confirms centralized agent management is working and all endpoints are shipping telemetry to the SIEM.

---

## Kali Password Spray Attack Rule

![Kali Password Spray Attack Rule](Kali%20Password%20Spray%20Attack%20Rule.png)

Shows the custom threshold detection rule created in Elastic Security. The rule queries for Event ID 4625 (failed logon) from Kali's IP address (`192.168.1.227`), grouped by `host.name`, with a threshold of 5+ failed attempts. Severity is set to High with a risk score of 73, mapped to MITRE ATT&CK T1110.003.

---

## Elastic Alert Dashboard

![Elastic Alert Dashboard](Elastic%20Alert%20Dashboard%202.png)

Shows the Elastic Security alerts dashboard with HIGH severity alerts fired across all three Windows machines — `win-c2kjctd2otq`, `victim1`, and `victim2` — as a result of the password spray attack. Confirms the detection pipeline is working end to end.

---

## Kali Password Spray Attack Detections

![Kali Password Spray Attack Detections](Kali%20Password%20Spray%20Attack%20Detections.png)

Shows all Kali Password Spray Attack alerts grouped by host, confirming detections fired separately for each targeted machine. Demonstrates the `host.name` group-by field working correctly in the threshold rule.

---

## Kali Password Spray Attack on Windows Server

![Kali Password Spray Attack on Windows Server](Kali%20Password%20Spray%20Attack%20on%20Windows%20Server%20.png)

Shows the Kali Linux attacker machine (`192.168.1.227`) running `smbclient` password spray commands against the Windows Server 2022 Domain Controller (`192.168.1.200`) over SMB port 445. Each command attempts authentication with a different wrong password, generating Event ID 4625 failed logon events on the target machine that are captured by the Elastic Agent and shipped to the SIEM.

---

## Kali Password Spray Attack on Victim 1

![Kali Password Spray Attack on Victim1](Kali%20Password%20Spray%20Attack%20on%20Victim1.png)

Shows the Kali Linux attacker machine (`192.168.1.227`) running `smbclient` password spray commands against Windows 11 Victim 1 (`192.168.1.238`) over SMB port 445. Each failed authentication attempt generates Event ID 4625 on Victim 1, which is collected by the Elastic Agent and forwarded to the SIEM where the detection rule fires.

---

## Kali Password Spray Attack on Victim 2

![Kali Password Spray Attack on Victim2](Kali%20Password%20Spray%20Attack%20on%20Victim2%20.png)

Shows the Kali Linux attacker machine (`192.168.1.227`) running `smbclient` password spray commands against Windows 11 Victim 2 (`192.168.1.231`) over SMB port 445. Each failed authentication attempt generates Event ID 4625 on Victim 2, which is collected by the Elastic Agent and forwarded to the SIEM where the detection rule fires.

---

## Windows Server — Failed Logon Logs

![Windows Server logon failed logs](Windows%20Server%20logon%20failed%20logs%20.png)

Shows raw Windows Security Event ID 4625 (Failed Logon) logs from the Windows Server 2022 Domain Controller in Kibana Discover. Logs show `source.ip: 192.168.1.227` (Kali), the targeted username, NTLM authentication package, and logon type 3 (network logon). This is the raw evidence the detection rule fires on.

---

## Victim 1 — Failed Logon Logs

![Victim 1 logon failed logs](Victim%201%20logon%20failed%20logs%20.png)

Shows raw Event ID 4625 logs from Windows 11 Victim 1 in Kibana Discover. Confirms failed authentication attempts from Kali's IP address were captured and shipped to the SIEM by the Elastic Agent.

---

## Victim 2 — Failed Logon Logs

![Victim 2 logon failed logs](Victim%202%20logon%20failed%20logs%20.png)

Shows raw Event ID 4625 logs from Windows 11 Victim 2 in Kibana Discover. Confirms failed authentication attempts from Kali's IP address were captured and shipped to the SIEM by the Elastic Agent.
