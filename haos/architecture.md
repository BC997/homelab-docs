# Home Assistant OS: As-Built Reference

## Host

| Property | Value |
|---|---|
| Hostname | homeassistant.local |
| IP | 10.0.0.8 |
| Proxmox node | Cupcake |
| VM ID | 112 |
| CPU | 8c |
| RAM | 16GB |
| Disk | 32G |

## External access

HAOS is accessible externally via https://ha.<DOMAIN> through NPM.
See networking/remote-access.md for full NPM and Cloudflare configuration.

## Active integrations

| Integration | Type | Notes |
|---|---|---|
| UniFi Protect | Official | G4 Doorbell Pro, IP direct, SSL verification disabled |
| UniFi Network | Official | UDM-SE gateway |
| Google Nest | Official | Nest thermostat |
| Samsung SmartThings | Official | Washer and dryer |
| Dyson | HACS (cmgrayb/hass-dyson) | Two Dyson purifiers |
| Synology NAS | Official | Primary and secondary NAS units |
| Open-Meteo | Official | Weather data, entity: weather.home |
| WiFi Power Pet Door | HACS (ha-powerpetdoor) | IP-based, local |
| Plex | Official | Webhook-based automations |

## Key entities

| Entity | Description |
|---|---|
| sensor.thermostat_temperature | Current indoor temperature |
| climate.thermostat | Nest thermostat control |
| weather.home | Outdoor weather via Open-Meteo |
| notify.mobile_app_* | Mobile push notifications |

## Active automations

| Automation | Trigger | Action |
|---|---|---|
| Window open suggestion | Outdoor temp < indoor temp while AC cooling (template edge + hourly time_pattern) | Push notification |
| NAS SMART health alert | Synology SMART status change | Push notification |
| Proxmox VM/node down | Alertmanager webhook | Push notification |
| Plex playing | Plex webhook (play event) | Push notification |
| Plex buffering/paused | Plex webhook (buffer/pause event) | Push notification |
| Plex resumed | Plex webhook (resume event) | Push notification |

Plex webhooks use trigger.data.payload | from_json (multipart/form-data format).

## HTTP config

Configured to accept proxied traffic from NPM. Set in configuration.yaml:

    http:
      ip_ban_enabled: true
      login_attempts_threshold: 2
      use_x_forwarded_for: true
      trusted_proxies:
        - 10.0.0.1
        - 10.0.0.0/24

No SSL configured inside HAOS. NPM handles all SSL termination.

## Pending projects

| Project | Status | Notes |
|---|---|---|
| Z-Wave lock integration | Pending hardware purchase | Fingerprint scan -> webhook -> HAOS -> lock.unlock |
| Robot vacuum integration | Pending hardware purchase | Local integration via MQTT, Valetudo |
| Z-Wave USB stick | Pending | On Cupcake -> ZWaveJS2MQTT Docker on cli-docker -> HAOS via TCP |
