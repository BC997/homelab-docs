# RBT-CLI Migration

## Overview

Migration of RBT (VM 107, rbt.lab.local, 10.0.0.5) from Ubuntu Desktop to a
clean Ubuntu Server CLI VM (VM 111, rbt-cli.lab.local, 10.0.0.9) on Echo node.
This completes the CLI-first homelab philosophy established with plex-cli (VM 109).

## VM Details

| | Old RBT | New RBT-CLI |
|---|---|---|
| VM ID | 107 | 111 |
| Hostname | rbt.lab.local | rbt-cli.lab.local |
| IP | 10.0.0.5 | 10.0.0.9 |
| OS | Ubuntu Desktop | Ubuntu Server (VM 100 template) |
| Node | Echo | Echo |
| Status | Hibernated | Active |

## Puppet Role

`role::media_server` → `profile::base` + `profile::nordvpn` + `profile::media_client` + `profile::nfs_media`

## NordVPN Configuration

NordVPN installed via official .deb repo (not snap). Key lessons learned:

- Snap install creates empty/masked systemd unit — must remove `/lib/systemd/system/nordvpn.service` before reinstalling via deb
- NordVPN 5.0.0 pushes `~.` DNS domain via nordlynx interface, overriding local DNS
- Fix: `/etc/systemd/network/nordlynx.network` drop-in with `DefaultRoute=no` and empty `DNS`/`Domains`
- LAN subnet allowlist conflicts with LAN Discovery — disable LAN Discovery first
- Required allowlist entries: subnet 10.0.0.x/24, ports 22/53/8140
- Auto-connect set to Netherlands
- Login via token stored in eyaml at `profile::nordvpn::token` in common.eyaml

## Media Client Migration

Config files migrated from old RBT:
- `/home/bcsb/.config/media-client/media-client.conf` — WebUI address updated from .147 to .245
- `/home/bcsb/.local/share/media-client/BT_backup/` — Media files and fast-resume data

WebUI: http://10.0.0.9:<PORT>
Bound to nordlynx interface, save path /mnt/nas/plexmediaserver/Media/

## NFS Mount

Managed by `profile::nfs_media` — 10.0.0.x:/volume1/PlexMediaServer → /mnt/nas/plexmediaserver

## Pending

- Delete VM 107 (target: late June 2026)
- Update rbt-puppet-remediation.md — snap-based nordvpn on old RBT is now superseded by deb install on rbt-cli

## NordVPN Duplicate Daemon Fix

NordVPN deb on Ubuntu 24.04 has two service units — `nordvpn.service` (SysV init.d)
and `nordvpnd.service` (native systemd). Both start on boot causing duplicate daemons.

Fix applied:
- Disabled `nordvpn.service` (SysV) via Puppet exec
- Switched Puppet service resource to `nordvpnd.service` and `nordvpnd.socket`
- Added `nordvpn-kill-duplicate-daemon` exec with `unless` guard:
  `[ $(pgrep -c nordvpnd) -le 1 ]`
- Do NOT kill nordvpnd PIDs directly — causes VM reboot/shutdown

## Additional Fix — init.d Script Removal

`systemd-sysv-generator` auto-starts `/etc/init.d/nordvpn` even when the service
is disabled, spawning a second nordvpnd on every boot. Fix: rename the init.d
script to `/etc/init.d/nordvpn.bak` via Puppet `exec` with `onlyif` guard.
Also updated `media-client@bcsb` drop-in from `nordvpn.service` to
`nordvpnd.service` to match the new service unit.
