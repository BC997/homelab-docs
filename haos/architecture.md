# Home Assistant OS: As Built Reference

## Host

| Property | Value |
|---|---|
| IP | 10.0.0.13 |
| Proxmox | VM 112 on Butternut |
| Disks | Shared NFS storage, so the VM can move under HA |

## External access

Available at `https://ha.<DOMAIN>` through NPM. See `networking/remote-access.md`.

## Integrations

| Integration | Source | Notes |
|---|---|---|
| UniFi Protect | Official | G4 Doorbell Pro with fingerprint reader |
| UniFi Network | Official | UDM SE |
| Google Nest | Official | Thermostat |
| SmartThings | Official | Washer and dryer |
| Dyson | HACS | Two purifiers |
| Synology DSM | Official | Both NAS units |
| Open-Meteo | Official | Outdoor temperature |
| Power Pet Door | HACS | Local, IP based |
| Plex | Webhooks | Playback automations |

## Automations

| Automation | Trigger | Action |
|---|---|---|
| Window open suggestion | Outdoor temp drops below indoor temp while cooling | Push notification |
| NAS health alert | Synology SMART status change | Push notification |
| Proxmox VM or node down | Alertmanager webhook | Push notification |
| Plex playback | Plex webhook events | Push notification |

Plex webhooks arrive as multipart form data, so templates read `trigger.data.payload | from_json`.

## HTTP and proxy settings

HAOS accepts traffic from NPM with forwarded headers and bans IPs after repeated failed logins. Trusted proxy settings were moved out of configuration.yaml and into the UI under Settings > System > Network. No TLS is configured inside HAOS. NPM terminates it.

## Pending projects

| Project | Plan |
|---|---|
| Z-Wave smart lock | Fingerprint scan on the doorbell, webhook to HAOS, unlock |
| Z-Wave controller | USB stick on a Proxmox node, Z-Wave JS in Docker, connected to HAOS over TCP |
| Robot vacuum | Rooted vacuum running Valetudo, local MQTT into HAOS, no cloud |
| Health dashboard | Smart ring data through Apple Health into InfluxDB and Grafana |
