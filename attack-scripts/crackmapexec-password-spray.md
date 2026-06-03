# CrackMapExec SMB Password Spray — Phase 2

## Overview

Password spray attack simulated from the Kali Linux attacker VM against both domain-joined Windows 11 endpoints using CrackMapExec over SMB (port 445).

**MITRE ATT&CK:** T1110.003 — Password Spraying  
**Tactic:** Credential Access  
**Tool:** CrackMapExec  
**Protocol:** SMB (port 445)  
**Attacker:** 192.168.1.227 (Kali)  
**Targets:** victim1.lab.local (192.168.1.251) and victim2.lab.local (192.168.1.231)

---

## Command

```bash
crackmapexec smb 192.168.1.251 192.168.1.231 \
  -u administrator labuser guest \
  -p Password123! password123 Welcome1 Summer2026! Winter2026! Passw0rd! \
  --smb-timeout 30
```

## Flags Explained

| Flag | Purpose |
|---|---|
| `smb` | Target protocol |
| `-u` | Usernames to spray |
| `-p` | Passwords to try against each username |
| `--smb-timeout 30` | Extended timeout to handle NETBIOS handshake latency on the rebuilt victim1 VM |

---

## Result

All attempts returned `STATUS_LOGON_FAILURE` — no successful authentications. Both endpoints generated `DeviceLogonEvents` in Microsoft Defender for Endpoint, which were ingested into Microsoft Sentinel and triggered the Kali Password Spray Attack analytics rule.

---

## Detection

Both Elastic Stack and Microsoft Sentinel detected this attack independently. Sentinel fired 34 alerts across multiple 5-minute detection windows covering both victim1 and victim2.
