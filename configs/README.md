# Configs

This folder contains configuration files used in the SOC Home Lab.

---

## Sysmon Configuration

**File:** `sysmon-config.xml`

The SwiftOnSecurity Sysmon configuration used on all three Windows machines — Windows Server 2022, Victim 1, and Victim 2. This config provides deep endpoint telemetry including:

- Event ID 1 — Process creation with full command line and hashes
- Event ID 3 — Network connections with source and destination IPs
- Event ID 11 — File creation
- Event ID 13 — Registry value modifications

The config was deployed on each Windows machine using:

```powershell
sysmon -accepteula -i sysmon-config.xml
```

Original config source: https://github.com/SwiftOnSecurity/sysmon-config

---

## Detection Rule — Kali Password Spray Attack

**File:** `detection-rule.ndjson`

Custom threshold detection rule exported from Elastic Security. Detects password spray attacks by monitoring for 5 or more failed logon attempts (Event ID 4625) from a single source IP, grouped by host.

**Rule summary:**

| Setting | Value |
|---|---|
| Rule type | Threshold |
| Query | `event.code: "4625" and source.ip: "192.168.1.227"` |
| Group by | `host.name` |
| Threshold | 5 failed logons |
| Severity | High |
| Risk score | 73 |
| MITRE ATT&CK | T1110.003 — Password Spraying |

To import this rule into Elastic Security go to **Security → Rules → Import rules** and upload the file.
