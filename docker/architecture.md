# Docker Architecture: As Built Reference

## Host

| Property | Value |
|---|---|
| Hostname | cli-docker.lab.local |
| IP | 10.0.0.11 |
| Proxmox | VM 110 on Cupcake |
| OS | Ubuntu Server |

## Overview

cli-docker is the main Docker host. Compose stacks live under `~/docker/`. Puppet manages the host and renders the compose files for most stacks from templates, so a rebuilt host comes back with the same services. Watchtower handles image updates on a weekly schedule (see below).

## Container inventory

| Container | Purpose |
|---|---|
| npm | Reverse proxy and TLS termination |
| pihole | Local DNS and ad blocking |
| portainer | Docker management UI |
| watchtower | Weekly image updates (maintained fork) |
| prometheus | Metrics collection |
| grafana | Dashboards |
| loki | Log aggregation |
| promtail | Log shipping |
| influxdb | Time series storage |
| alertmanager | Alert routing |
| unifi-poller | UniFi metrics |
| pve-exporter | Proxmox metrics |
| gitea | Self hosted git |
| vaultwarden | Password manager |
| seerr | Media requests |
| tautulli | Plex analytics |
| radarr, sonarr, bazarr | Media library management |
| audiobookshelf | Audiobook server |
| cloudflare-ddns | Dynamic DNS for external hostnames |

## Puppet on cli-docker

cli-docker runs the agent with `role::standard_server` plus service profiles, including `profile::docker`, `profile::arr_stack`, `profile::seerr`, `profile::tautulli`, `profile::pihole_env`, `profile::prometheus_env`, and `profile::nfs_media`. Secrets for these profiles live in the node's eyaml file.

## Watchtower

The original Watchtower project was archived in December 2025. It kept working here only because its compose file pinned Docker API 1.45, and the host's Docker Engine was already on API 1.57. In November 2026 it was replaced with the maintained fork, `nickfedor/watchtower`, which negotiates the API version itself, so the pin was removed.

| Setting | Value |
|---|---|
| Schedule | Sundays 04:00 |
| Cleanup of old images | On |
| Stopped containers | Skipped |
| Label filter | None, so every container updates unless labeled `com.centurylinklabs.watchtower.enable=false` |
| Compose file | Hand managed, not rendered by Puppet |

The swap was done after a monitor only dry run with the fork (`--run-once --monitor-only`), which scanned every container, found the pending updates, and changed nothing. Only the Watchtower container itself restarted.

Watchtower sends its notifications to a Home Assistant webhook through a shoutrrr generic URL with `template=json`. A custom notification template always sends a one line summary, `Scanned N, updated N, failed N`, followed by the names of anything updated or failed, so a quiet week still checks in. Home Assistant raises a separate alert if no report arrives within two hours of the scheduled run.

The fork logs `Update session completed` where the original logged `Session done`.

## Pi-hole and NPM port note

NPM owns port 80 on this host, so the Pi-hole admin UI runs on a different port.
