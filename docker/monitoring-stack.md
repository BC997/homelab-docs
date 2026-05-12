# Monitoring Stack: As-Built Reference

## Overview

The monitoring stack runs on cli-docker (10.0.0.3) as Docker Compose services
under ~/docker/monitoring/. It collects metrics from all Proxmox nodes, VMs, UniFi
hardware, and lab services.

## Components

| Service | Purpose | Port | UI |
|---|---|---|---|
| Prometheus | Metrics collection and storage | 9090 | http://10.0.0.3:9090 |
| Grafana | Dashboards and visualization | 3000 | http://10.0.0.3:3000 |
| Loki | Log aggregation | 3100 | via Grafana |
| InfluxDB | Time-series storage (secondary) | 8086 | http://10.0.0.3:8086 |
| Alertmanager | Alert routing and notification | 9093 | http://10.0.0.3:9093 |
| UniFi Poller | UniFi metrics exporter | 9130 | - |
| pve-exporter | Proxmox metrics exporter | 9221 | - |
| Promtail | Log shipping agent | - | - |

## Exporters

### Node Exporter

Node Exporter runs on every VM in the fleet via profile::node_exporter (part of
profile::base). It exposes host-level metrics (CPU, memory, disk, network) on port 9100.

Hosts scraped by Prometheus:

| Host | IP | Port |
|---|---|---|
| puppet.lab.local | 10.0.0.2 | 9100 |
| cli-docker.lab.local | 10.0.0.3 | 9100 |
| plex.lab.local | 10.0.0.4 | 9100 |
| rbt.lab.local | 10.0.0.5 | 9100 |
| puppetdb.lab.local | 10.0.0.6 | 9100 |

### pve-exporter

Scrapes Proxmox API for node and VM metrics across all 5 cluster nodes.
Credentials stored in eyaml in node-specific Hiera data.

### UniFi Poller

Scrapes UniFi controller for network device metrics.
Credentials stored in eyaml.

## Alertmanager

Alertmanager receives alerts from Prometheus and routes notifications.
HAOS receives Proxmox VM and node down alerts via Alertmanager webhook.

Webhook endpoint in HAOS handles:
- Proxmox VM down alerts
- Proxmox node down alerts

## Grafana

Grafana connects to both Prometheus and InfluxDB as data sources.
Loki is configured as a log data source for log exploration alongside metrics.
Promtail runs on cli-docker and ships logs to Loki.

Default dashboards cover:
- Proxmox node and VM resource usage (via pve-exporter)
- Per-VM host metrics (via Node Exporter)
- UniFi network device stats (via UniFi Poller)
