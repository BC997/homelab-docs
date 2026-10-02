# Networking Architecture: As Built Reference

## Gateway

| Property | Value |
|---|---|
| Device | UniFi UDM SE |
| IP | 10.0.0.1 |
| OS version | UniFi OS 5.0.16 |
| Firewall rules | Settings > Policy Engine > Traffic and Firewall Rules |

Older UniFi docs point to Settings > Firewall & Security. That path does not exist in UniFi OS 5.0.16.

## Networks

| Network | Subnet | Purpose |
|---|---|---|
| Default | 10.0.0.0/24 | Main LAN, all lab infrastructure |
| IoT VLAN | 10.0.1.0/24 | Smart home devices |

IoT devices are placed on the IoT VLAN with a per client Virtual Network Override.

## Firewall rules

| Rule | Source | Destination | Action |
|---|---|---|---|
| Block IoT to LAN | IoT VLAN | Default | Block |
| Allow IoT to WAN | IoT VLAN | Internet | Allow |

IoT devices reach the internet but cannot start connections into the main LAN. The main LAN can still reach IoT devices.

A firewall rule was also added after a WAN scan from a Kali VM showed the gateway's DNS resolver answering on port 53 from outside.

## Key LAN hosts

| Host | IP | Role |
|---|---|---|
| Gateway | 10.0.0.1 | Router |
| openvox-cli.lab.local | 10.0.0.10 | OpenVox primary, OpenVoxDB, PuppetBoard |
| cli-docker.lab.local | 10.0.0.11 | Docker host, NPM, Pi-hole, Gitea, monitoring |
| plex-cli.lab.local | 10.0.0.12 | Plex Media Server |
| HAOS | 10.0.0.13 | Home Assistant OS |
| Primary NAS | 10.0.0.20 | Media and VM storage |
| Secondary NAS | 10.0.0.21 | Backup storage |
| Apricot | 10.0.0.101 | Proxmox node |
| Butternut | 10.0.0.102 | Proxmox node |
| Cupcake | 10.0.0.103 | Proxmox node |
| Dessert | 10.0.0.104 | Proxmox node |

## Local DNS

Pi-hole v6 on cli-docker answers for every `lab.local` name. All nodes, VMs, and services have static DHCP reservations in UniFi and a matching Pi-hole Local DNS record. See `docker/pihole.md`.

Browsers with Secure DNS (DNS over HTTPS) turned on skip Pi-hole and fail to resolve `lab.local` names. Turn it off on lab workstations.
