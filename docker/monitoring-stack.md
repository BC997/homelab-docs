# Monitoring Stack: As Built Reference

## Overview

The monitoring stack runs on cli-docker under `~/docker/monitoring/` and covers Proxmox nodes, VMs, UniFi hardware, and lab services.

| Service | Purpose | Port |
|---|---|---|
| Prometheus | Metrics collection | 9090 |
| Grafana | Dashboards | 3000 |
| Loki | Log aggregation | 3100 |
| Promtail | Log shipping | n/a |
| InfluxDB | Time series storage | 8086 |
| Alertmanager | Alert routing | 9093 |
| UniFi Poller | UniFi metrics | 9130 |
| pve-exporter | Proxmox metrics | 9221 |

## Node Exporter

Every Puppet managed VM runs Node Exporter on port 9100 through `profile::base`, so a new VM is scraped as soon as its first agent run finishes and Prometheus is told about it.

| Host | IP |
|---|---|
| openvox-cli.lab.local | 10.0.0.10 |
| cli-docker.lab.local | 10.0.0.11 |
| plex-cli.lab.local | 10.0.0.12 |

## pve-exporter

Scrapes the Proxmox API for node and VM metrics across all four nodes. Credentials are in eyaml.

## Alerting

Alertmanager sends ProxmoxVMDown and NodeDown alerts to a Home Assistant webhook, which pushes notifications to an iPhone and iPad.

## Grafana

Data sources are Prometheus, InfluxDB, and Loki. Dashboards cover Proxmox resources, per VM host metrics, and UniFi devices.
