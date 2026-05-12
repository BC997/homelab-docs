# Networking Architecture: As-Built Reference

## Gateway

| Property | Value |
|---|---|
| Device | UniFi UDM-SE |
| IP | 10.0.0.1 |
| OS version | UniFi OS 5.0.16 |
| Firewall rules location | Settings -> Policy Engine -> Traffic and Firewall Rules |

Note: Older UniFi documentation references Settings -> Firewall & Security.
That path does not exist in UniFi OS 5.0.16. All rules are under Policy Engine.

## Networks

| Network | Subnet | Purpose |
|---|---|---|
| Default | 10.0.0.0/24 | Main LAN — all lab infrastructure |
| IoT VLAN | 10.0.1.0/24 | IoT — smart home devices |

IoT devices are assigned to the IoT VLAN via Virtual Network Override per client.

## Firewall rules

Two rules are active in Policy Engine:

| Rule | Source | Destination | Action |
|---|---|---|---|
| Block IoT to LAN | IoT VLAN | Default | Block (always) |
| Allow IoT to WAN | IoT VLAN | Internet | Allow (always) |

IoT devices can reach the internet but cannot initiate connections to the main LAN.
Main LAN devices can reach IoT devices (unidirectional block).

## Key LAN hosts

| Host | IP | Role |
|---|---|---|
| Gateway | 10.0.0.1 | Router |
| Primary NAS | 10.0.0.20 | Media storage |
| Secondary NAS | 10.0.0.21 | Backup storage |
| cli-docker.lab.local | 10.0.0.3 | Docker host, NPM, Pi-hole |
| HAOS | 10.0.0.8 | Home Assistant OS |
| Plex | 10.0.0.4 | Plex Media Server |
| RBT | 10.0.0.5 | Download VM |
| Apricot (Proxmox) | 10.0.0.101 | Proxmox node |
| Butternut (Proxmox) | 10.0.0.102 | Proxmox node |
| Cupcake (Proxmox) | 10.0.0.103 | Proxmox node |
| Dessert (Proxmox) | 10.0.0.104 | Proxmox node |
| Echo (Proxmox) | 10.0.0.105 | Proxmox node |
| puppet.lab.local | 10.0.0.2 | Puppet primary |
| puppetdb.lab.local | 10.0.0.6 | PuppetDB + PuppetBoard |

## Local DNS

Pi-hole v6 running on cli-docker provides local DNS for all lab hosts.
All lab services are registered under the lab.local convention.

Pi-hole API authentication uses X-FTL-SID header (v6 change from prior versions).
Write operations use: PUT /api/config/dns/hosts/{IP}%20{domain}

All Proxmox nodes, VMs, and services have static DHCP reservations.
