# INE-Network-Based-Attacks-CTF-1

Hands-on notes from the INE Network-Based Attacks lab, covering diagnostic service discovery, SNMP management enumeration, and anonymous SMB share access.

## Lab Tasks

1. Get the perimeter host to release its diagnostic token.
2. Interrogate the host's management layer.
3. Enumerate the host's file services without credentials.

## Environment

- **Target hostname:** `target1.ine.local`
- **Target IP:** `192.23.186.3`
- **Host name reported by services:** `BR-EDGE01`
- **Workgroup:** `ASHCROFT`
- **Tools recorded:** Nmap, Ncat, `onesixtyone`, `snmpwalk`, `smbclient`

## Summary of Findings

- Nmap identified SSH, SMB/NetBIOS, SNMP/SMUX, and a custom diagnostic service on TCP port `13337`.
- The diagnostic service's response indicated that it expected a management source port. Ncat with local source port `53` returned the Task 1 diagnostic token.
- SNMP enumeration exposed standard system details and an application-specific entry containing a token and an operational note.
- Anonymous SMB enumeration revealed a `public` share. A downloaded staff notice contained the Task 3 perimeter self-check token.

## Repository Contents

- [`lab-notes.md`](lab-notes.md) — detailed command-by-command notes and explanations.

## Publication and Safety Note

This repository is intended as a learning record for an authorized training environment. Flag values and the SNMP community string are redacted. The internal-server pivot task is not documented as solved because no completed command/output transcript was provided for it. Before publishing a detailed INE Skill Check solution publicly, review INE's current rules on sharing lab solutions and flags.
