# Lab Notes — INE Network-Based Attacks CTF 1

## Scope

These notes reflect the transcript supplied for the authorized INE lab. The target was `target1.ine.local` (`192.23.186.3`), which reported the host name `BR-EDGE01` and workgroup `ASHCROFT`.

Flag values and the SNMP community string are redacted. Task 5 was discussed afterward, but the supplied transcript does not contain a completed solution for the internal-server pivot, so this document does not invent one.

---

## Task 1: Get the perimeter host to release its diagnostic token

### 1. Scan all TCP ports

```bash
nmap -sV -p- target1.ine.local
```

- `nmap` runs the network scan.
- `-sV` probes open ports to identify services and possible versions.
- `-p-` scans TCP ports 1–65535.
- `target1.ine.local` is the target hostname; it resolved to `192.23.186.3` in this run.

Relevant results:

- `22/tcp`: SSH — OpenSSH 9.6p1.
- `139/tcp` and `445/tcp`: SMB/NetBIOS — Samba.
- `199/tcp`: SMUX, associated with SNMP management.
- `13337/tcp`: custom/unrecognized diagnostic service.

The service fingerprint for port `13337` included:

```text
NETDIAG: access denied (diagnostic channel expects mgmt source port)
```

This response suggested that the service expected a particular client source port.

### 2. Connect using source port 53

```bash
sudo ncat -nv -p 53 192.23.186.3 13337
```

- `sudo` runs Ncat with elevated privileges; binding to a low-numbered source port may require this.
- `ncat` is a network connection utility from Nmap.
- `-n` disables name resolution.
- `-v` enables verbose output.
- `-p 53` sets the local source port to `53`.
- `192.23.186.3` is the destination host.
- `13337` is the destination TCP port.

The service returned a diagnostic banner and a token. The token value is redacted.

**Lesson:** inspect service responses carefully. The banner identified a source-port condition that informed the next connection attempt.

---

## Task 2: Interrogate the host's management layer

### 1. Discover an SNMP community string

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt 192.23.186.3
```

- `onesixtyone` checks candidate SNMP community strings against a target.
- `-c` specifies the candidate list file.
- The path is the SecLists list used in the recorded run.
- `192.23.186.3` is the target.

The output identified a community string that received an SNMP response. The value is redacted in this public version.

### 2. Read standard system information

```bash
snmpwalk -v1 -c [REDACTED_COMMUNITY] 192.23.186.3 1.3.6.1.2.1.1
```

- `snmpwalk` walks the OIDs beneath the specified subtree.
- `-v1` selects SNMP version 1.
- `-c [REDACTED_COMMUNITY]` stands in for the discovered community string; the actual value is not published.
- `192.23.186.3` is the target.
- `1.3.6.1.2.1.1` is the standard SNMP system group subtree.

The output exposed the system description, contact, hostname, and physical location. The hostname was `BR-EDGE01`, and the contact field was `noc@ashcroft.local`.

### 3. Query the application-specific subtree

```bash
snmpwalk -v1 -c [REDACTED_COMMUNITY] 192.23.186.3 1.3.6.1.4.1.8072.1.3.2.3
```

The returned entries included a diagnostic token and a maintenance note. The token and community string are redacted. The note referred to the `svc_backup` service account and nightly backups over SMB and SSH.

**Lesson:** SNMP may expose operational details in addition to basic system metadata. Application-specific OIDs deserve review during an authorized assessment.

---

## Task 3: Enumerate file services without credentials

### 1. List SMB shares without a password

```bash
smbclient -L //192.23.186.3 -N
```

- `smbclient` is a command-line SMB/CIFS client.
- `-L` lists the shares offered by the server.
- `//192.23.186.3` specifies the SMB server.
- `-N` suppresses the password prompt, allowing an anonymous/null-password attempt.

The output listed:

- `public` — public staff notices.
- `backup` — backup operators only.
- `IPC$` — SMB inter-process communication share.

The workgroup was `ASHCROFT`.

### 2. Connect to the public share anonymously

```bash
smbclient //192.23.186.3/public -N
```

The connection succeeded in the recorded session. At the SMB prompt, the following command listed the share contents:

```text
ls
```

The share contained `notice_to_staff.txt`.

### 3. Download and inspect the notice

At the SMB prompt:

```text
get notice_to_staff.txt
exit
```

- `get notice_to_staff.txt` downloads the remote file to the local working directory.
- `exit` closes the SMB client session.

Back in the Kali shell:

```bash
cat notice_to_staff.txt
```

`cat` prints the downloaded file. The notice stated that the `public` share was world-readable and warned staff not to put sensitive files there. It also contained the Task 3 perimeter self-check token, redacted here.

**Lesson:** verify actual share permissions rather than assuming that share names or comments enforce security.

---


## Key Takeaways

1. Full TCP port scans can reveal important services on nonstandard ports.
2. Service banners and error messages can provide useful enumeration clues.
3. SNMP can disclose host identity, contact details, location, and application-specific data.
4. Anonymous SMB access can expose internal documents and tokens.
5. Document only actions and findings supported by the actual lab transcript.

## Scope and Publication Reminder

Use these techniques only in an authorized lab or assessment. Check INE's current publication policy before making detailed Skill Check solutions public.
