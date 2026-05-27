# Plex Media Server: As-Built Reference
## Overview
Plex Media Server runs on a dedicated VM (plex.lab.local). It is managed by Puppet
via profile::plex. Media is served from a Synology NAS via NFS mount.

## Host
| Property | Value |
|---|---|
| Hostname | plex.lab.local |
| IP | 10.0.0.155 |
| Proxmox node | Echo |
| VM ID | 101 |
| CPU | 16c |
| RAM | 64GB |
| Puppet role | role::media_server |

## External Access
| Property | Value |
|---|---|
| URL | https://plex.example.com |
| Route | NPM on cli-docker → Plex VM :32400 |
| plex.tv relay | Disabled |

## NFS Mount
| Property | Value |
|---|---|
| NFS share | 10.0.0.35:/volume1/PlexMediaServer |
| Mount point | /mnt/nas/plexmediaserver |
| Mount mode | read-only |

## APT Repository
Plex changed their repository structure with PMS 1.43.0. The new repo is at
repo.plex.tv and requires a new GPG key.

| Property | Value |
|---|---|
| Repo file | /etc/apt/sources.list.d/plex.list |
| GPG key | /etc/apt/keyrings/plexmediaserver.v2.gpg |
| Key URL | https://downloads.plex.tv/plex-keys/PlexSign.v2.key |
| Repo URL | https://repo.plex.tv/deb/ public main |

Old repo (downloads.plex.tv) was removed May 25 2026 during migration to 1.43.2.

## Puppet
Managed by profile::plex. Handles:
- GPG key installation
- Apt repo configuration
- Package installation
- Daily upgrade cron at 3am
- VM reboot cron at 4am
- systemd override for auto-restart on failure

## Updates
Puppet cron runs apt-get install --only-upgrade plexmediaserver daily at 3am.
Logs at /var/log/plex-upgrade.log on the Plex VM.
VM reboots at 4am via root crontab.
