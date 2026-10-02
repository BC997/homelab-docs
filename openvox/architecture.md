# OpenVox Architecture: As Built Reference

## Primary

| Property | Value |
|---|---|
| Role | OpenVox primary (server, OpenVoxDB, PuppetBoard, agent) |
| Hostname | openvox-cli.lab.local |
| IP | 10.0.0.10 |
| Proxmox VM | 101 on Apricot |
| OS | Ubuntu Server |
| Version | OpenVox 8.x |
| PuppetBoard | http://10.0.0.10:8000 |

OpenVox is the community maintained fork of Open Source Puppet. The lab moved to it after Open Source Puppet packaging was discontinued. File locations stayed the same through the migration. See `migration.md`.

The old split layout (separate Puppet primary and PuppetDB VMs) is retired. Everything now runs on one primary.

## Code paths

| Path | Purpose |
|---|---|
| /etc/puppet/code/environments/production/ | Deployed environment (r10k target, never hand edit) |
| /etc/puppet/code/environments/production/manifests/site.pp | Node classification |
| /etc/puppet/code/environments/production/modules/ | Roles, profiles, component modules |
| /etc/puppet/code/environments/production/data/ | Hiera data |
| /etc/puppet/code/environments/production/hiera.yaml | Hiera config |
| ~/repos/homelab-puppet on the primary | Working clone where all edits happen |

Anything edited under the deployed path is wiped on the next r10k deploy. See `r10k-workflow.md`.

## Hiera hierarchy

Highest precedence first:

1. data/nodes/%{trusted.certname}.eyaml (per node, encrypted)
2. data/nodes/%{trusted.certname}.yaml (per node, plain)
3. data/common.eyaml (lab wide, encrypted)
4. data/common.yaml (lab wide, plain)

Lab wide overrides were consolidated into common.yaml during the migration. Per node files only hold what is truly specific to that node.

## Roles and profiles

Every node gets exactly one role. Roles compose profiles. Profiles manage resources.

| Role | Assigned to | Profiles |
|---|---|---|
| role::standard_server | openvox-cli, cli-docker, node default | profile::base |
| role::media_server | plex-cli | profile::base, profile::plex |

cli-docker also includes service profiles that render its compose stacks (see `docker/architecture.md`).

### profile::base

Applied everywhere. Highlights:

* Base packages including qemu-guest-agent
* SSH hardening, fail2ban, and auditd
* ufw with default deny inbound, LAN allowed, and per node exceptions from Hiera
* Node Exporter for Prometheus
* Timezone set to America/Los_Angeles with an idempotent exec guarded by `unless`, so cron schedules mean the same thing on every host
* Agent and OpenVoxDB client config, parameterized so a node can point at a different server through Hiera
* Nightly reboot cron, scheduled and toggled through Hiera

### Nightly reboot

The reboot window lives in Hiera. Nodes that must stay up, like the primary, opt out per node:

    profile::base::manage_nightly_reboot: false

The profile has an explicit `ensure => absent` branch for that flag. Simply not managing the cron when the flag is false would leave an already applied cron in place.

## Node classification gotcha

A node's certname must exactly match a node block in site.pp. If it does not, the node silently falls through to:

    node default { include role::standard_server }

It gets the base profile only, and no error is raised. A new VM with a typo in its hostname will look healthy while missing its role.
