# Plex Media Server: As-Built Reference

## Overview

Plex Media Server runs on a dedicated headless CLI VM (plex-cli). It is managed
by Puppet via profile::plex. Media is served from a Synology NAS via NFS mount.
Migrated from a desktop Ubuntu VM (VM 101, now hibernated) to a CLI Ubuntu Server
VM (VM 109) for reduced resource overhead and improved Puppet management.

## Host

| Property | Value |
|---|---|
| Hostname | plex-cli.lab.local |
| IP | 10.0.0.4 |
| Proxmox node | Echo |
| VM ID | 109 |
| CPU | 16c |
| RAM | 64GB |
| Puppet role | role::media_server |
| Base template | VM 100 (Ubuntu-ServerCLI-Template) |

## Previous Host (hibernated)

| Property | Value |
|---|---|
| VM ID | 101 |
| State | Hibernated, target delete late June 2026 |
| OS | Ubuntu Desktop |

## External Access

| Property | Value |
|---|---|
| URL | https://plex.<DOMAIN> |
| Route | NPM on cli-docker → plex-cli :32400 |
| plex.tv relay | Disabled |

## NFS Mount

| Property | Value |
|---|---|
| NFS share | <NAS_IP>:/volume1/PlexMediaServer |
| Mount point | /mnt/nas/plexmediaserver |
| Mount mode | read-only |
| Managed by | profile::plex (Puppet) |

Do not modify, move, or delete any data on this NFS mount.

## Installation

Plex installed via official deb package (not snap). Migrated from snap to deb
during the CLI VM rebuild to ensure clean Puppet management and avoid snap
dependency issues.

## APT Repository

Plex changed their repository structure with PMS 1.43.0. The new repo is at
repo.plex.tv and requires a new GPG key.

| Property | Value |
|---|---|
| Repo file | /etc/apt/sources.list.d/plex.list |
| GPG key | /etc/apt/keyrings/plexmediaserver.v2.gpg |
| Key URL | https://downloads.plex.tv/plex-keys/PlexSign.v2.key |
| Repo URL | https://repo.plex.tv/deb/ public main |

Old repo (downloads.plex.tv) was removed during migration to 1.43.2.

## Puppet

Managed by profile::plex. Handles:
- GPG key installation
- Apt repo configuration
- Package installation
- NFS mount via profile::nfs_media
- Daily upgrade cron at 3am
- VM reboot cron at 4am
- systemd override for auto-restart on failure

## Updates

Puppet cron runs apt-get install --only-upgrade plexmediaserver daily at 3am.
Logs at /var/log/plex-upgrade.log on the Plex VM.
VM reboots at 4am via root crontab.
