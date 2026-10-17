# Splunk Universal Forwarder Installer

Scripts that install the Splunk Universal Forwarder and point it at your indexer's IP and receiving port.

| Script | OS | Collects |
|---|---|---|
| `install_uf.sh` | Linux (Debian/Ubuntu, RHEL family) | auth.log / secure, syslog / messages, audit.log (if present) |
| `install_uf.ps1` | Windows | Security, System, Application event logs; Sysmon and PowerShell Operational if present |

## Before you run

1. On the Splunk server: **Settings → Forwarding and receiving → Configure receiving** → add port `9997`.
2. Open that port in the server's firewall (and Proxmox VM firewall / network ACLs if used).
3. Get the forwarder package link: splunk.com → Universal Forwarder download → pick `.tgz` (Linux) or `.msi` (Windows) → **Copy wget link**. Splunk changes these URLs every release, so the scripts take the URL as an argument.
4. If you use an index other than `main`, create it on the server first.

## Linux

```bash
chmod +x install_uf.sh
sudo ./install_uf.sh -i 192.168.1.50 -p 9997 -u '<tgz-url>'
# or with a local file and a custom index
sudo ./install_uf.sh -i 192.168.1.50 -f ./splunkforwarder-*.tgz -x linux
```

| Flag | Meaning | Default |
|---|---|---|
| `-i` | Splunk server IP (required) | — |
| `-p` | Receiving port | `9997` |
| `-u` / `-f` | Package URL or local `.tgz` path (one required) | — |
| `-a` | Forwarder admin password (or env `SPLUNK_PASSWORD`; prompts otherwise) | prompt |
| `-x` | Destination index | `main` |

## Windows (admin PowerShell)

```powershell
Set-ExecutionPolicy -Scope Process Bypass
.\install_uf.ps1 -ServerIP 192.168.1.50 -Port 9997 -MsiUrl "<msi-url>"
# or with a local file and a custom index
.\install_uf.ps1 -ServerIP 192.168.1.50 -MsiPath .\splunkforwarder.msi -Index windows
```

## Verify

On the Splunk server:

```
index=_internal host=<endpoint-hostname> | head 10
index=main host=<endpoint-hostname> | stats count by sourcetype
```

If `_internal` shows the host, the forwarder is connected. If not, check `splunkd.log` on the endpoint
(`/opt/splunkforwarder/var/log/splunk/` or `C:\Program Files\SplunkUniversalForwarder\var\log\splunk\`).

## Notes

- The Linux service runs as a dedicated `splunkfwd` user, added to `adm` so it can read auth/syslog on Debian/Ubuntu. `audit.log` is root-only by default; the script prints how to grant access.
- The Windows install runs as Local System so the Security log is readable — fine for a lab, tighten for production.
- Watch your license: on Splunk Free (500 MB/day), Security + Sysmon logs from several machines add up fast.
