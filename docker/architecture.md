# Docker Architecture: As-Built Reference

## Host

| Property | Value |
|---|---|
| Hostname | cli-docker.lab.local |
| IP | 10.0.0.3 |
| Proxmox node | Dessert |
| VM ID | 110 |
| OS | Ubuntu 22.04 |
| Username | ubuntu |
| CPU | 8c |
| RAM | 8GB |
| Disk | 64G |

## Overview

cli-docker is the primary Docker host for the lab. All compose stacks live under
~/docker/ and are managed via Docker Compose. Watchtower handles automatic container
updates. Puppet manages system-level config on this host but does not manage containers.

## Container inventory

| Container | Image | Port(s) | Purpose |
|---|---|---|---|
| npm | jc21/nginx-proxy-manager | 80, 81, 443 | Reverse proxy, SSL termination |
| pihole | pihole/pihole | 53, 80 | Local DNS, ad blocking |
| portainer | portainer/portainer-ce | 9000 | Docker management UI |
| watchtower | containrrr/watchtower | - | Automatic container updates |
| prometheus | prom/prometheus | 9090 | Metrics collection |
| grafana | grafana/grafana | 3000 | Metrics dashboards |
| loki | grafana/loki | 3100 | Log aggregation |
| promtail | grafana/promtail | - | Log shipping to Loki |
| influxdb | influxdb:2.7 | 8086 | Time-series storage |
| alertmanager | prom/alertmanager | 9093 | Alert routing |
| unifi-poller | ghcr.io/unpoller/unpoller | 9130 | UniFi metrics exporter |
| pve-exporter | prompve/prometheus-pve-exporter | 9221 | Proxmox metrics exporter |
| audiobookshelf | ghcr.io/advplyr/audiobookshelf | 90 | Audiobook server |
| gitea | gitea/gitea | 3001, 2222 | Self-hosted git |

## Watchtower

Watchtower polls for updated container images and restarts containers automatically.
It manages all containers on cli-docker. No manual image updates needed.

## Puppet on cli-docker

cli-docker runs the Puppet agent and is assigned role::standard_server.
Certname: cli-docker.lab.local (serial s3, signed by CA-C).
Puppet manages system-level config only — not containers.

## Gitea

Self-hosted Gitea runs as a container on cli-docker (port 3001).
Accessible at http://gitea.lab.local:3001.
