# Splunk Universal Forwarder Installer

Scripts that install the Splunk Universal Forwarder and point it at your indexer's IP and receiving port.

By default they download **Universal Forwarder 10.6.1.1** straight from Splunk (Linux amd64/arm64 auto-detected, Windows x64), so the only required input is your server IP. For a different version, copy the download link from splunk.com (Universal Forwarder → your OS → **Copy wget link**) and pass it with `-u` / `-MsiUrl`.

| Script | OS | Collects |
|---|---|---|
| `install_uf.sh` | Linux (Debian/Ubuntu, RHEL family) | auth.log / secure, syslog / messages, audit.log (if present) |
| `install_uf.ps1` | Windows | Security, System, Application event logs; Sysmon and PowerShell Operational if present |

## Before you run

1. On the Splunk server: **Settings → Forwarding and receiving → Configure receiving** → add port `9997`.
2. Open that port in the server's firewall (and Proxmox VM firewall / network ACLs if used).
3. If you use an index other than `main`, create it on the server first.

## Linux

```bash
chmod +x install_uf.sh
sudo ./install_uf.sh -i 192.168.1.50
# custom port and index
sudo ./install_uf.sh -i 192.168.1.50 -p 9997 -x linux
# a specific package URL, or a file you already downloaded
sudo ./install_uf.sh -i 192.168.1.50 -u '<tgz-url>'
sudo ./install_uf.sh -i 192.168.1.50 -f ./splunkforwarder-*.tgz
```

| Flag | Meaning | Default |
|---|---|---|
| `-i` | Splunk server IP (required) | — |
| `-p` | Receiving port | `9997` |
| `-u` / `-f` | Package URL or local `.tgz` path | UF 10.6.1.1 for this CPU |
| `-a` | Forwarder admin password (or env `SPLUNK_PASSWORD`; prompts otherwise) | prompt |
| `-x` | Destination index | `main` |

## Windows (admin PowerShell)

```powershell
Set-ExecutionPolicy -Scope Process Bypass
.\install_uf.ps1 -ServerIP 192.168.1.50
# a specific MSI URL
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
