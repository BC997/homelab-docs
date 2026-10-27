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

cli-docker also includes service profiles that render its compose stacks (see `docker/architecture.md`). openvox-cli adds `profile::openvox_primary`, which enforces the server and PuppetBoard fixes (see `puppetboard-fixes.md`).

### profile::base

Applied everywhere. Highlights:

* Base packages including qemu-guest-agent
* SSH hardening, fail2ban, and auditd
* ufw with default deny inbound, LAN allowed, and per node exceptions from Hiera
* Node Exporter for Prometheus
* Timezone set to America/Los_Angeles with an idempotent exec guarded by `unless`, so cron schedules mean the same thing on every host
* Agent and OpenVoxDB client config, parameterized so a node can point at a different server through Hiera
* Nightly reboot cron, scheduled and toggled through Hiera
* The primary's name pinned in `/etc/hosts` on every node, so agent runs don't depend on Pi-hole
* Time sync: `systemd-timesyncd` kept enabled and running

### Nightly reboot

The reboot window lives in Hiera. Nodes that must stay up, like the primary, opt out per node:

    profile::base::manage_nightly_reboot: false

The profile has an explicit `ensure => absent` branch for that flag. Simply not managing the cron when the flag is false would leave an already applied cron in place.

The time is a Hiera setting too, 5:00 by default. The Docker host runs Pi-hole, so it reboots 30 minutes earlier, and DNS is back up before the Plex host restarts:

    profile::base::nightly_reboot_hour: '4'
    profile::base::nightly_reboot_minute: '30'

### Agent runs and DNS

Every node used to find the primary through Pi-hole. Agent runs that landed while the Docker host was rebooting failed with "Temporary failure in name resolution," and each one raised a failure alert. `base::dns` now adds the primary to `/etc/hosts` on every node with a `host` resource, so an agent run reaches the server whether Pi-hole is up or not. The name and address are Hiera parameters, defaulting to `openvox-cli.lab.local` and 10.0.0.10.

## Node classification gotcha

A node's certname must exactly match a node block in site.pp. If it does not, the node silently falls through to:

    node default { include role::standard_server }

It gets the base profile only, and no error is raised. A new VM with a typo in its hostname will look healthy while missing its role.
