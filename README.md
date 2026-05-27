# homelab-docs
Sanitized as-built reference documentation for a self-hosted homelab. All docs reflect the current state of the environment.

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
