# homelab-docs

A self hosted homelab built from the ground up with a focus on privacy, local first design, and hands on engineering across infrastructure, security, and automation. This repository is a sanitized public snapshot of the internal lab documentation. It exists to show real engineering work, not theory.

## Why it was built

The lab closes the gap between knowing how enterprise infrastructure works and actually building and running it. Every component was chosen on purpose, configured from scratch, and is actively maintained. When something breaks, it gets debugged, root caused, and written down. The goal is production discipline on consumer hardware.

## What's in it

The lab runs on a four node Proxmox VE 9 cluster. Configuration is managed by OpenVox 8, the community fork of Open Source Puppet, with a role and profile structure, Hiera data separation, r10k for environment deploys, and eyaml for secrets. OpenVoxDB and PuppetBoard give visibility into facts, catalogs, reports, and drift across the fleet. Gitea is the self hosted source of truth, and this GitHub repo is a sanitized snapshot of it. The configuration code itself is published in [homelab-openvox](https://github.com/BC997/homelab-openvox).

Networking runs on a UniFi UDM SE with a segmented IoT VLAN and a single reverse proxy entry point using DNS validated TLS certificates.

Observability comes from Prometheus and Grafana covering cluster health, network telemetry, and host metrics, with Loki and Promtail for logs and Alertmanager routing alerts into Home Assistant.

Home automation runs on Home Assistant OS with local integrations across security, climate, appliances, and presence.

## Recent work

Since the last snapshot the lab went through several large changes.

* **Upgraded the cluster from Proxmox VE 8 to 9 with no downtime for core services.** All four nodes were on 8.4.0 from March 2025, about 239 packages behind and past end of life. A phased plan cleared an unused Ceph cluster, caught every node up to the last 8.4 release, pinned NIC names, then upgraded one node at a time to Debian 13 and Proxmox VE 9.2 on kernel 7.0, using HA maintenance mode so the OpenVox primary, Home Assistant and DNS moved live. See `proxmox/pve9-upgrade.md`.
* **Made sure the nodes never fall behind again.** Debian security updates now install daily on every node, Proxmox updates follow a monthly playbook with a Home Assistant reminder, a weekly vzdump job backs up every VM, and Proxmox notifications reach the phone instead of an unread mailbox. Old snapshots and orphaned disks from a removed node were cleaned up, about 75 GB.
* **Replaced an archived container updater.** Watchtower had been archived for almost a year and only worked because of a pinned Docker API version. It was swapped for the maintained fork after a dry run, and every weekly run now reports to Home Assistant, with a separate alert if a run goes missing. See `docker/architecture.md`.
* **Found an automation that had been failing silently.** When HACS became Home Assistant's built in Marketplace, a missing entity made the weekly update check fail every Sunday with no visible error. Fixed, and alerts now stay open in Alert history until they are checked off by hand. See `haos/architecture.md`.
* **Migrated from Open Source Puppet to OpenVox.** Stood up a new primary with OpenVoxDB and PuppetBoard, moved every managed node over, fixed a set of non obvious configuration gaps, then retired the old Puppet primary and PuppetDB hosts. See `openvox/migration.md`.
* **Tightened certificate signing.** Autosign moved from a wildcard to an explicit allowlist of certnames.
* **Root caused a full cluster reboot.** A NIC transmit ring hang on one node dropped corosync, and HA watchdog fencing rebooted every HA active node at once. Fixed the NIC, documented the failure chain, and changed how quorum margin is kept. See `proxmox/lessons-learned.md`.
* **Shrank the cluster from five nodes to four.** Removed one node cleanly, moved its workloads, and repurposed the hardware as a standalone workstation.
* **Standardized fleet behavior in code.** Pacific timezone, a Hiera controlled nightly reboot window with per node opt out, and aligned Proxmox update timers.
* **Made VMs portable.** HA managed VMs now use the shared bridge so they can move between nodes.
* **Added services.** Seerr, Tautulli, and Bazarr as Puppet managed compose stacks, plus Vaultwarden and Cloudflare DDNS.
* **Made the primary heal itself.** The hand applied OpenVoxDB and PuppetBoard fixes are now enforced by a profile. An upgrade that undoes one gets repaired on the next agent run. Tested by breaking a fix on purpose. See `openvox/puppetboard-fixes.md`.
* **Removed every access token.** r10k pulls with a read only SSH deploy key, edits push with their own SSH key, and the database moved off its default password. Every user of the old token was found and moved before it was revoked.
* **Audited the cluster network.** Confirmed every node runs the NIC model behind the August outage, with zero errors and no repeat hangs. Traced a pattern of short corosync drops to shared LAN traffic rather than hardware. Every VM now sits on the shared bridge, so any VM can run on any node.
* **Alerted on drift, not just recorded it.** A checker on the primary reads OpenVoxDB every 5 minutes and sends every failed run, and every change made outside a deploy window, to Home Assistant as a phone alert with the full report log. The open source server leaves `corrective_change` empty, so drift is told apart from planned change by timing against the last r10k deploy. See `openvox/drift-alerts.md`.
* **Rebuilt Home Assistant around one dashboard and real alerts.** Built with Claude through an approval gated MCP connection: a nine page dashboard, a utility bill breakdown, alerts for lab hosts, the car, laundry, and the fridge, a 7 day alert history, and weekly self updates with a backup before each one. See `haos/architecture.md`.
* **Fixed lab DNS on a VPN workstation.** The VPN client deliberately drops DNS to the local network. Lab names now resolve through the hosts file instead of weakening the VPN. See `networking/workstation-dns.md`.

## Roadmap

* Add CI to the code repo: lint, syntax check, YAML lint, and secret scanning on every push
* Rebuild a cloud init VM template so new VMs no longer need a full install
* Give corosync its own second network link, and alert on link drops in the meantime
* Stand up Ansible alongside OpenVox for provisioning, Proxmox host management, and one off remediation, with OpenVox kept for ongoing state enforcement
* Automate Pi-hole local DNS records, which are still a manual step

## Structure

### openvox/
| Doc | Contents |
|---|---|
| architecture.md | Primary host, code paths, Hiera hierarchy, roles and profiles, node classification |
| migration.md | Open Source Puppet to OpenVox migration and the fixes it took |
| puppetboard-fixes.md | Server and PuppetBoard fixes, now enforced and self healing |
| r10k-workflow.md | Change workflow from edit to verified apply |
| binaries.md | Binary paths and common commands |
| ca-and-agent-bootstrap.md | Autosign allowlist, onboarding, and cert cleanup |
| eyaml-secrets.md | Keypair locations, encrypted files, encryption workflow |
| drift-alerts.md | Phone alerts for failed runs and drift outside a deploy |

### proxmox/
| Doc | Contents |
|---|---|
| cluster.md | Nodes, quorum, VM placement, HA rules, NIC tuning, patching, backups, alerts |
| lessons-learned.md | Quorum, fencing, portability, and other lessons from real incidents |
| pve9-upgrade.md | Proxmox VE 8 to 9 rolling upgrade: plan, problems, and the patching and alerting that came out of it |

### networking/
| Doc | Contents |
|---|---|
| architecture.md | Gateway, networks, firewall rules, LAN host reference |
| remote-access.md | NPM, Cloudflare DNS and DDNS, NAT loopback |
| workstation-dns.md | Resolving lab names on a workstation whose VPN blocks LAN DNS |
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
| architecture.md | Integrations, dashboard, energy breakdown, alerts, weekly updates, AI assisted administration, pending projects |

## Notes

* IPs and domains are sanitized. Real values use a private range and a private domain.
* Credentials, tokens, CA fingerprints, and keys are never committed here.
