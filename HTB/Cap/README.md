# Hack The Box — Cap

> **Platform:** Hack The Box  
> **Operating system:** Linux  
> **Difficulty:** Easy  
> **Status:** User and root access obtained  
> **Primary themes:** Web authorization, PCAP analysis, plaintext FTP credentials, password reuse, Linux capabilities

## Summary

Cap is a Linux machine that demonstrates how several weaknesses can be chained into full compromise. Service enumeration identified FTP, SSH, and a web-based Security Dashboard. Testing the dashboard's numeric scan IDs revealed access to scan data beyond the current scan. One accessible PCAP contained FTP authentication traffic in plaintext. The recovered credentials worked over SSH for the user `nathan`. Local enumeration then revealed an unsafe `cap_setuid` capability on `/usr/bin/python3.8`, which was used to obtain UID 0.

This walkthrough records the investigation flow and why each observation mattered. Sensitive values such as passwords and flag contents are intentionally omitted.

## Attack-chain overview

```text
Nmap
  |
  v
Security Dashboard (HTTP)
  |
  v
Manipulate /data/<scan_id>
  |
  v
Access a sensitive PCAP
  |
  v
Wireshark: FTP authentication in cleartext
  |
  v
Credential reuse against SSH
  |
  v
Shell as nathan --> user flag
  |
  v
getcap -r /
  |
  v
Python with cap_setuid
  |
  v
UID 0 --> root flag
```

## 1. Reconnaissance

Initial scan:

```bash
nmap -sC -sV <TARGET_IP>
```

The scan identified three open TCP ports:

| Port | Service | Observed version / banner |
|---:|---|---|
| 21/tcp | FTP | vsftpd 3.0.3 |
| 22/tcp | SSH | OpenSSH 8.2p1 Ubuntu |
| 80/tcp | HTTP | Gunicorn; Security Dashboard |

The web application was a useful next step because it exposed a feature named **Security Snapshot**.

Detailed notes: [Enumeration/Nmap.md](Enumeration/Nmap.md)

## 2. Web enumeration and IDOR

Running Security Snapshot redirected the browser to a route of the form:

```text
/data/<scan_id>
```

Different scan IDs were requested manually. The username displayed in the UI stayed the same, but the underlying scan statistics changed (for example, one capture showed 13 packets while another showed 0). This was evidence that the ID selected the scan record, and led to the access-control investigation.

The issue is consistent with **Insecure Direct Object Reference (IDOR) / broken object-level authorization**: the server should check whether the current user is authorized to access the requested scan, rather than relying on an identifier alone.

Detailed notes: [Exploitation/IDOR.md](Exploitation/IDOR.md)

## 3. PCAP analysis

A capture associated with scan ID `0` contained useful traffic. It was downloaded from the dashboard and inspected in Wireshark. Following the relevant TCP stream revealed FTP authentication commands, including a username and a password sent without transport encryption. The credential itself is not reproduced here.

The protocol observed was **FTP**. Standard FTP does not protect the login exchange with encryption, so a party who can capture the traffic may recover credentials.

Detailed notes: [Exploitation/PCAP-Analysis.md](Exploitation/PCAP-Analysis.md)

## 4. Initial access through SSH

The recovered credential was tested against SSH using the username observed in the capture:

```bash
ssh nathan@<TARGET_IP>
```

Authentication succeeded, providing an interactive shell as `nathan`.

The user flag was located in `/home/nathan/user.txt`. The flag value is omitted from this public write-up.

## 5. Privilege escalation via Linux capabilities

From the `nathan` shell, local capability enumeration was performed:

```bash
getcap -r / 2>/dev/null
```

Relevant output:

```text
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
```

The key finding was `cap_setuid` on the Python interpreter. This capability allows a process to set its user ID, making UID 0 reachable in this misconfigured context.

The capability was abused in the lab:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Verification:

```bash
id
whoami
```

The `id` output confirmed `uid=0(root)`. The root flag was then accessed at `/root/root.txt`; its value is intentionally omitted.

Detailed notes: [Privilege-Escalation/Linux-Capabilities.md](Privilege-Escalation/Linux-Capabilities.md)

## Findings and defensive recommendations

| Finding | Impact | Recommended mitigation |
|---|---|---|
| Scan ID authorization flaw | A user may access another user's scan records or PCAPs | Enforce server-side object-level authorization on every scan request and download |
| Sensitive PCAP exposure | Captures may contain confidential traffic and credentials | Restrict access, minimize capture scope and retention, and protect stored captures |
| FTP authentication in cleartext | Network observers may recover credentials | Replace FTP with SFTP or FTPS and avoid transmitting secrets over unencrypted channels |
| Password reuse across FTP and SSH | One exposed password can grant access to a second service | Use unique credentials, credential management, and appropriate SSH authentication controls |
| Risky Python file capability | A low-privileged user can escalate to root | Remove unnecessary capabilities; audit file capabilities and use least privilege |

## Lessons learned

1. A numeric object ID is not an authorization control.
2. PCAP files can be sensitive evidence and should be protected like credentials.
3. Credential reuse can turn a limited disclosure into interactive access.
4. Linux capabilities need to be audited; interpreter binaries with powerful capabilities are especially risky.
5. Each step should be supported by observable evidence, not assumptions.

## Evidence and notes

- [Nmap enumeration](Enumeration/Nmap.md)
- [IDOR investigation](Exploitation/IDOR.md)
- [PCAP analysis](Exploitation/PCAP-Analysis.md)
- [Linux capabilities](Privilege-Escalation/Linux-Capabilities.md)
- [Command reference](Notes/commands.md)
- [Screenshots folder](Screenshots/README.md)

## Disclaimer

This is a personal educational write-up of a Hack The Box lab. Only test systems where you have explicit authorization. Do not publish flags, passwords, or other secrets in public documentation.
