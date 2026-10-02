# homelab-docs

A self hosted homelab built from the ground up with a focus on privacy, local first design, and hands on engineering across infrastructure, security, and automation. This repository is a sanitized public snapshot of the internal lab documentation. It exists to show real engineering work, not theory.

## Why it was built

The lab closes the gap between knowing how enterprise infrastructure works and actually building and running it. Every component was chosen on purpose, configured from scratch, and is actively maintained. When something breaks, it gets debugged, root caused, and written down. The goal is production discipline on consumer hardware.

## What's in it

The lab runs on a four node Proxmox 8 cluster. Configuration is managed by OpenVox 8, the community fork of Open Source Puppet, with a role and profile structure, Hiera data separation, r10k for environment deploys, and eyaml for secrets. OpenVoxDB and PuppetBoard give visibility into facts, catalogs, reports, and drift across the fleet. Gitea is the self hosted source of truth, and this GitHub repo is a sanitized snapshot of it. The configuration code itself is published in [homelab-openvox](https://github.com/BC997/homelab-openvox).

Networking runs on a UniFi UDM SE with a segmented IoT VLAN and a single reverse proxy entry point using DNS validated TLS certificates.

Observability comes from Prometheus and Grafana covering cluster health, network telemetry, and host metrics, with Loki and Promtail for logs and Alertmanager routing alerts into Home Assistant.

Home automation runs on Home Assistant OS with local integrations across security, climate, appliances, and presence.

## Recent work

Since the last snapshot the lab went through several large changes.

* **Migrated from Open Source Puppet to OpenVox.** Stood up a new primary with OpenVoxDB and PuppetBoard, moved every managed node over, fixed a set of non obvious configuration gaps, then retired the old Puppet primary and PuppetDB hosts. See `openvox/migration.md`.
* **Tightened certificate signing.** Autosign moved from a wildcard to an explicit allowlist of certnames.
* **Root caused a full cluster reboot.** A NIC transmit ring hang on one node dropped corosync, and HA watchdog fencing rebooted every HA active node at once. Fixed the NIC, documented the failure chain, and changed how quorum margin is kept. See `proxmox/lessons-learned.md`.
* **Shrank the cluster from five nodes to four.** Removed one node cleanly, moved its workloads, and repurposed the hardware as a standalone workstation.
* **Standardized fleet behavior in code.** Pacific timezone, a Hiera controlled nightly reboot window with per node opt out, and aligned Proxmox update timers.
* **Made VMs portable.** HA managed VMs now use the shared bridge so they can move between nodes.
* **Added services.** Seerr, Tautulli, and Bazarr as Puppet managed compose stacks, plus Vaultwarden and Cloudflare DDNS.

## Roadmap

* Codify the hand applied PuppetBoard fixes so they survive upgrades and rebuilds
* Move r10k from HTTP token auth to an SSH deploy key
* Stand up Ansible alongside OpenVox for provisioning, Proxmox host management, and one off remediation, with OpenVox kept for ongoing state enforcement
* Automate Pi-hole local DNS records, which are still a manual step

## Structure

### openvox/
| Doc | Contents |
|---|---|
| architecture.md | Primary host, code paths, Hiera hierarchy, roles and profiles, node classification |
| migration.md | Open Source Puppet to OpenVox migration and the fixes it took |
| puppetboard-fixes.md | PuppetBoard and server fixes applied after the migration |
| r10k-workflow.md | Change workflow from edit to verified apply |
| binaries.md | Binary paths and common commands |
| ca-and-agent-bootstrap.md | Autosign allowlist, onboarding, and cert cleanup |
| eyaml-secrets.md | Keypair locations, encrypted files, encryption workflow |

### proxmox/
| Doc | Contents |
|---|---|
| cluster.md | Nodes, quorum, VM placement, NIC tuning, update timers |
| lessons-learned.md | Quorum, fencing, portability, and other lessons from real incidents |

### networking/
| Doc | Contents |
|---|---|
| architecture.md | Gateway, networks, firewall rules, LAN host reference |
| remote-access.md | NPM, Cloudflare DNS and DDNS, NAT loopback |
| nfs.md | NFS share, mounts, Synology permissions |

### docker/
| Doc | Contents |
|---|---|
| architecture.md | Container inventory and Puppet managed stacks |
| gitea.md | Repo inventory and r10k integration |
| pihole.md | Pi-hole v6 API auth and local DNS |
| monitoring-stack.md | Prometheus, Grafana, Loki, InfluxDB, Alertmanager, exporters |
| plex.md | Plex VM, NFS mount, apt repo, Puppet management |
| seerr.md | Media requests |
| tautulli.md | Plex analytics |

### haos/
| Doc | Contents |
|---|---|
| architecture.md | Integrations, automations, HTTP config, pending projects |

## Notes

* IPs and domains are sanitized. Real values use a private range and a private domain.
* Credentials, tokens, CA fingerprints, and keys are never committed here.
