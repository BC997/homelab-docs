# Docker Architecture: As Built Reference

## Host

| Property | Value |
|---|---|
| Hostname | cli-docker.lab.local |
| IP | 10.0.0.11 |
| Proxmox | VM 110 on Cupcake |
| OS | Ubuntu Server |

## Overview

cli-docker is the main Docker host. Compose stacks live under `~/docker/`. Puppet manages the host and renders the compose files for most stacks from templates, so a rebuilt host comes back with the same services. Watchtower handles image updates on a weekly schedule.

## Container inventory

| Container | Purpose |
|---|---|
| npm | Reverse proxy and TLS termination |
| pihole | Local DNS and ad blocking |
| portainer | Docker management UI |
| watchtower | Weekly image updates |
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
| radarr, sonarr, lidarr, bazarr | Media library management |
| audiobookshelf | Audiobook server |
| cloudflare-ddns | Dynamic DNS for external hostnames |

## Puppet on cli-docker

cli-docker runs the agent with `role::standard_server` plus service profiles, including `profile::docker`, `profile::arr_stack`, `profile::seerr`, `profile::tautulli`, `profile::pihole_env`, `profile::prometheus_env`, and `profile::nfs_media`. Secrets for these profiles live in the node's eyaml file.

## Pi-hole and NPM port note

NPM owns port 80 on this host, so the Pi-hole admin UI runs on a different port.
