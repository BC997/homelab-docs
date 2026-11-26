# Home Assistant OS: As Built Reference

## Host

| Property | Value |
|---|---|
| IP | 10.0.0.13 |
| Proxmox | VM 112 on Butternut |
| Disks | Shared NFS storage, so the VM can move under HA |
| Versions | Core 2026.10, OS 18 |

## External access

Available at `https://ha.<DOMAIN>` through NPM. See `networking/remote-access.md`.

## Integrations

| Integration | Source | Notes |
|---|---|---|
| UniFi Protect | Official | G4 Doorbell Pro with fingerprint reader, plus PoE cameras |
| UniFi Network | Official | UDM SE |
| Google Nest | Official | Thermostat |
| SmartThings | Official | Washer, dryer, fridge, TV |
| Dyson | HACS | Two purifiers |
| Synology DSM | Official | Both NAS units, drive SMART status |
| Open-Meteo | Official | Outdoor temperature |
| Power Pet Door | HACS | Local, IP based |
| Plex | Official and webhooks | Now playing and library sensors; webhooks for playback automations |
| Teslemetry | Official | Car status, tires, battery health, charging. Read only use, so no paid command credits |
| Ping (ICMP) | Official | Reachability of every Proxmox node and lab VM |
| Local To-do | Official | Holds the 7 day Alert history |
| HA-MCP | HACS | Lets Claude manage HA over the LAN under approval rules (see below) |

## Dashboard

A custom dashboard called Home Hub replaced the default Overview. It uses one glass style built from shared button-card templates inside the dashboard config, so every page looks the same without a theme file.

| Page | What it shows |
|---|---|
| Home | Lights, TV, appliance cost, pet door, inside vs outside temperature, buttons to every other page |
| Climate | Thermostat mode and range, air quality from both purifiers, climate trends |
| Laundry and kitchen | Washer, dryer, fridge |
| Pets | Pet door controls, including whether the dogs can go out |
| Security | Camera grid with recent activity below |
| Media | Plex streams playing now (cover art, user, device, progress), living room TV and soundbar |
| Lab and NAS | Both NAS units (storage, drive health, temperatures, CPU, memory, network, DSM), Proxmox nodes and VMs, internet |
| Utility | Monthly power and gas bills split across what HA can measure or estimate |
| Car | Status, controls, tires, battery health, driving, charging, trips, safety |

### Energy breakdown

The utility does not offer an Opower link, so monthly bills are entered by hand in four helpers (kWh, power cost, therms, gas cost). An effective rate is derived from them, and each slice is priced at that rate.

| Slice | Method |
|---|---|
| Fridge, washer, dryer | Measured by SmartThings |
| Car charging at home | Volts times amps from Teslemetry while charging at home, integrated to kWh |
| AC | Run hours times an estimated draw |
| Furnace and dryer gas | Run hours times the rated BTU input |
| Everything else | The rest of each bill |

## Automations

Every alert sends a phone notification and also saves an entry to an Alert history list. Entries stay open until they are checked off by hand, so the list works as a to do list of things that still need a look. A daily cleanup deletes checked off entries older than 7 days and never touches open ones.

| Automation | Trigger | Action |
|---|---|---|
| Lab host unreachable or back online | A Ping sensor goes down or comes back (about 30 seconds) | Push after 3 minutes down, with downtime on recovery. No skip windows: Patch Wednesday reboots finish in under 2 minutes, and a VM that doesn't come back still alerts |
| OpenVox drift and failures | Webhook from the OpenVox primary | Push with the full report log. See `openvox/drift-alerts.md` |
| Home Assistant back online | HA starts | Push, so a restart or host outage is visible afterward |
| Car tire pressure low | Any tire below 38 psi for 10 minutes | Push |
| Laundry done | Washer or dryer goes from run to stop | Push with cycle length |
| Fridge door left open | Open 2 minutes, then again on close | Push |
| Window open suggestion | Outdoor temp drops below indoor temp while cooling | Push |
| NAS health alert | Synology SMART status change | Push |
| Proxmox VM or node down | Alertmanager webhook | Push |
| Proxmox notifications | Webhook from the Proxmox notification system (backups, package updates, fencing, replication) | Push; title marks warnings and errors as problems |
| Monthly node updates | First Sunday of the month | Push, plus an open item to check off when the nodes are updated |
| Watchtower report | Webhook after every weekly container update run | Push with scanned, updated and failed counts; flags any failure. The startup message is skipped during the Docker host's Wednesday patch slot |
| Watchtower missed run | Two hours after the weekly run, if no report arrived | Push, plus an open item |
| Container check | Webhook from the Docker host, once per boot, when an auto restart container is still down 15 minutes after boot | Push, plus an open item listing the containers that are down. See `openvox/patch-wednesday.md` |
| Plex playback | Plex webhook events | Push |

Plex webhooks arrive as multipart form data, so templates read `trigger.data.payload | from_json`.

## Weekly updates

HA updates itself on Sundays, after the weekly container update. Proxmox node updates are monthly and done by hand. The VMs patch and reboot only if needed on Wednesday (see `openvox/patch-wednesday.md`). Each run notifies before it starts, and nothing runs when no update is waiting.

| Order | Updates | Safety |
|---|---|---|
| 1 | Marketplace (HACS) integrations and dashboard cards | Lists anything that failed; restarts HA only if an integration changed |
| 2 | HA Core | Backup first; skips .0 releases and betas, so it waits for the first patch release |
| 3 | HA OS | Backup first; skipped while a Core update is still running |

Supervisor updates itself. Apps stay manual.

Since HA 2026.11, HACS is the built in Marketplace and updates along with Core, so its old self update entity no longer exists. The weekly automation still listed it, and a state condition fails as a whole when any entity in its list is missing, so the automation had been skipping silently. The entity was removed from the list in November 2026.

## AI assisted administration

The dashboard and automations above were built with Claude through the HA-MCP custom component, working from a workstation over the LAN. Guardrails:

* The connection is turned on per session and off when the work is done.
* Destructive and high impact tools (remove, restart, backups, HACS, writing automations) wait for approval in the HA-MCP panel. Device control, cameras, files, and YAML editing are turned off.
* Claude reads and writes configuration only. Lights, locks, and other devices stay under manual control.
* A full HA backup and a Proxmox snapshot were taken before the first change and again at the end.

## HTTP and proxy settings

HAOS accepts traffic from NPM with forwarded headers and bans IPs after repeated failed logins. Trusted proxy settings were moved out of configuration.yaml and into the UI under Settings > System > Network. No TLS is configured inside HAOS. NPM terminates it.

## Pending projects

| Project | Plan |
|---|---|
| Proxmox VE integration | Built in integration with read only monitoring plus power buttons for the media VM only, then decide whether it replaces the Ping alerts |
| UPS monitoring | UPS data port to the main NAS, NAS UPS server, NUT in HA, rack power on the Utility page |
| Z-Wave smart lock | Fingerprint scan on the doorbell, webhook to HAOS, unlock |
| Z-Wave controller | USB stick on a Proxmox node, Z-Wave JS in Docker, connected to HAOS over TCP |
| Robot vacuum | Rooted vacuum running Valetudo, local MQTT into HAOS, no cloud |
| Health dashboard | Smart ring data through Apple Health into InfluxDB and Grafana |
