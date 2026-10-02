# Plex Media Server: As Built Reference

## Host

| Property | Value |
|---|---|
| Hostname | plex-cli.lab.local |
| IP | 10.0.0.12 |
| Proxmox | VM 109 on Dessert |
| OS | Ubuntu Server (headless) |
| Puppet role | role::media_server |

Plex moved from a desktop Ubuntu VM to this headless server VM to cut overhead and bring it fully under Puppet. The old VM has been removed.

## NFS

| Property | Value |
|---|---|
| Share | <NAS_IP>:/volume1/PlexMediaServer |
| Mount point | /mnt/nas/plexmediaserver |
| Mode | Read and write, so media can be removed from inside Plex |
| Managed by | profile::plex |

## Install and APT repo

Installed from the official deb, not the snap. Plex moved its repo with PMS 1.43.0.

| Property | Value |
|---|---|
| Repo file | /etc/apt/sources.list.d/plex.list |
| Key | /etc/apt/keyrings/plexmediaserver.v2.gpg |
| Repo | https://repo.plex.tv/deb/ public main |

## Puppet

`profile::plex` handles:

* GPG key and apt repo
* Package install
* Daily upgrade cron at 3 AM, logged to /var/log/plex-upgrade.log
* systemd override to restart on failure
* QEMU guest agent through the base profile

Nightly reboot behavior follows the fleet setting in `profile::base`, with a per node opt out in Hiera.

## External access

See `networking/remote-access.md`. The plex.tv relay is disabled.

## Home Assistant

Plex webhooks feed Home Assistant automations for playback started, paused, resumed, and buffering.
