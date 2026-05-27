# homelab-docs
A self-hosted homelab built from the ground up with a focus on privacy, local-first architecture, and hands-on engineering across infrastructure, security, and automation. This repository is a sanitized public snapshot of the internal as-built documentation. It exists to demonstrate real engineering work — not theoretical knowledge.

## Why it was built
The lab was built to close the gap between knowing how enterprise infrastructure works and having actually built and operated it. Every component was chosen deliberately, configured from scratch, and is actively maintained. When something breaks, it gets debugged and documented. The goal is a production-grade environment running on consumer hardware — with the same rigor applied to a real datacenter.

## What's in it
The lab runs on a 5-node Proxmox 8 cluster hosting 9 VMs and 21 container services across dedicated workload nodes. All configuration is managed via Open Source Puppet 8 with a full role/profile structure, Hiera data separation, r10k for environment management, and eyaml for secrets. Gitea serves as the self-hosted git backend and source of truth — this GitHub repo is a sanitized mirror.

Networking runs on a UniFi UDM-SE with VLAN-segmented IoT devices, IDS/IPS threat management, and a hardened remote access architecture. External services are exposed through a single reverse proxy entry point with DNS-validated TLS certificates — no unnecessary ports open.

Observability is handled by a full Prometheus and Grafana stack covering cluster health, network telemetry, and container metrics, with Loki and Promtail for log aggregation and Alertmanager for routing. PuppetDB and PuppetBoard provide configuration state visibility and drift detection across the fleet.

Home automation runs on Home Assistant OS managing 10+ integrations across security, climate, appliances, and presence detection — entirely local with no cloud dependency.

Security research is conducted against the lab's own infrastructure using Kali Linux, with a focus on identifying and closing real attack vectors.

## How it was built
Everything was built iteratively — starting with a single Proxmox node and expanding outward. Infrastructure is treated as code: no manual configuration that isn't tracked in version control. When a new service is added, it gets a Puppet profile, a compose template, and a doc. When something breaks, the root cause gets documented so it doesn't happen again. The runbooks and architecture docs in this repo reflect the actual state of the environment, not an idealized version of it.

## What this repo contains
Sanitized as-built reference documentation organized by domain. IPs and domains have been replaced with placeholder values. Credentials, CA fingerprints, and other sensitive data are never committed here.

## Structure

### puppet/
Open Source Puppet 8 infrastructure.
| Doc | Contents |
|---|---|
| architecture.md | Config path split, two puppet.conf files, Hiera hierarchy, role/profile structure |
| binaries.md | Binary paths, gem environments, r10k config, common commands, service management |
| ca-bootstrap.md | Active CA, cert paths, fleet cert inventory |
| r10k-workflow.md | Gitea -> r10k -> production environment deploy workflow |
| agent-rebootstrap.md | Per-host SSL re-bootstrap procedure |
| puppetdb-integration.md | PuppetDB wiring, storeconfigs gotcha, port reference |
| eyaml-secrets.md | PKCS7 keypair locations, encrypted files, encryption workflow |

### proxmox/
5-node Proxmox 8.4.0 cluster.
| Doc | Contents |
|---|---|
| cluster.md | Node IPs, full VM inventory, NIC tuning, onboarding workflow |

### networking/
UniFi UDM-SE, VLANs, firewall rules, remote access, and NFS.
| Doc | Contents |
|---|---|
| architecture.md | Gateway, networks, VLANs, firewall rules, full LAN host reference |
| remote-access.md | NPM, Cloudflare DNS, Pi-hole NAT loopback, HAOS HTTP config |
| nfs.md | NFS mount reference, fstab entries, Synology NFS and folder permissions |

### docker/
All Docker services running on cli-docker.
| Doc | Contents |
|---|---|
| architecture.md | Full container inventory, Watchtower, Puppet/Docker separation |
| gitea.md | Repo inventory, r10k integration, GitHub mirror reference |
| pihole.md | Pi-hole v6 API auth, DNS record management, local DNS reference |
| monitoring-stack.md | Prometheus, Grafana, Loki, InfluxDB, Alertmanager, exporters |
| plex.md | Plex VM, NFS mount, apt repo migration, Puppet management, external access |
| tautulli.md | Plex analytics, Puppet management, Plex token eyaml reference |

### haos/
Home Assistant OS.
| Doc | Contents |
|---|---|
| architecture.md | Integrations, automations, HTTP config, pending projects |

## Notes
- IPs and domains in this repo are sanitized. Real values use a private 192.168.x.x range.
- Private data (credentials, CA fingerprints, real IPs) is never committed here.
