# Cap — Nmap Enumeration

## Objective

Identify reachable TCP services and establish an initial attack surface.

## Command

```bash
nmap -sC -sV <TARGET_IP>
```

- `-sC`: run Nmap's default scripts.
- `-sV`: probe open ports to identify service/version information.

For a reusable saved report, run:

```bash
nmap -sC -sV -oN nmap_initial.txt <TARGET_IP>
```

## Observed results

| Port | State | Service | Version / detail |
|---:|---|---|---|
| 21/tcp | open | FTP | vsftpd 3.0.3 |
| 22/tcp | open | SSH | OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 |
| 80/tcp | open | HTTP | Gunicorn; page title: Security Dashboard |

Three TCP ports were open. The web application on port 80 was selected for further enumeration, while FTP and SSH remained relevant during credential investigation.

## Interpretation

- **FTP (21):** investigate anonymous access and whether credentials or sensitive files are exposed.
- **SSH (22):** potential interactive access if valid credentials are discovered.
- **HTTP (80):** inspect application functionality, URL patterns, and server-side authorization.

Do not infer exploitability from a banner alone; validate findings through controlled testing and evidence.
