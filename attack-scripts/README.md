# Password Spray Attack — T1110.003

## Overview

A password spray attack is a type of brute force attack where an attacker tries a small number of commonly used passwords against a large number of accounts. Unlike traditional brute force attacks that target a single account with many passwords, password spraying avoids account lockouts by spreading attempts across multiple accounts.

This technique is documented in the MITRE ATT&CK framework as **T1110.003 — Password Spraying**.

---

## Lab Environment

| Role | Machine | IP |
|---|---|---|
| Attacker | Kali Linux | 192.168.1.227 |
| Target 1 | Windows Server 2022 DC | 192.168.1.200 |
| Target 2 | Windows 11 Victim 1 | 192.168.1.238 |
| Target 3 | Windows 11 Victim 2 | 192.168.1.231 |

---

## Tool Used

**smbclient** — a command line tool for accessing SMB shares on Windows machines. Used here to simulate authentication attempts over SMB port 445.

---

## Commands Used

```bash
# Against Windows Server 2022 Domain Controller
smbclient //192.168.1.200/C$ -U administrator%wrongpassword1
smbclient //192.168.1.200/C$ -U administrator%wrongpassword2
smbclient //192.168.1.200/C$ -U administrator%wrongpassword3
smbclient //192.168.1.200/C$ -U labuser%wrongpassword1
smbclient //192.168.1.200/C$ -U labuser%wrongpassword2

# Against Windows 11 Victim 1
smbclient //192.168.1.238/C$ -U administrator%wrongpassword1
smbclient //192.168.1.238/C$ -U administrator%wrongpassword2
smbclient //192.168.1.238/C$ -U administrator%wrongpassword3
smbclient //192.168.1.238/C$ -U labuser%wrongpassword1
smbclient //192.168.1.238/C$ -U labuser%wrongpassword2

# Against Windows 11 Victim 2
smbclient //192.168.1.231/C$ -U administrator%wrongpassword1
smbclient //192.168.1.231/C$ -U administrator%wrongpassword2
smbclient //192.168.1.231/C$ -U administrator%wrongpassword3
smbclient //192.168.1.231/C$ -U labuser%wrongpassword1
smbclient //192.168.1.231/C$ -U labuser%wrongpassword2
```

---

## Evidence Generated

Each failed authentication attempt generated a Windows Security **Event ID 4625 — An account failed to log on** on the target machine, containing:

- `source.ip` — `192.168.1.227` (Kali Linux)
- `user.name` — the targeted account name
- `winlog.logon.type` — Network (type 3)
- `winlog.event_data.AuthenticationPackageName` — NTLM
- `host.name` — the targeted machine

---

## Detection

A custom threshold detection rule was created in Elastic Security:

- **KQL query:** `event.code: "4625" and source.ip: "192.168.1.227"`
- **Group by:** `host.name`
- **Threshold:** 5+ failed logons per host
- **Severity:** High — Risk score 73

**Result:** HIGH severity alerts fired for all three Windows machines.

---

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| Tactic | Credential Access |
| Technique | T1110 — Brute Force |
| Sub-technique | T1110.003 — Password Spraying |
| Platform | Windows |
| Data source | Windows Security Event Log |
