# Puppet Architecture: As-Built Reference

## Host

| Property | Value |
|---|---|
| Role | Puppet primary (server + agent) |
| Hostname | puppet.lab.local |
| IP | 10.0.0.2 |
| Proxmox node | Butternut |
| VM ID | 106 |
| OS | Ubuntu 22.04 |
| Username | puppet-homelab |
| Puppet version | Open Source Puppet 8 |

## The config path split

Two separate directory trees are in use. This is not a mistake — they serve different processes.

| Path | Owned by | Read by |
|---|---|---|
| /etc/puppetlabs/puppet/ | puppet (agent) | puppet agent CLI |
| /etc/puppet/puppetserver/conf.d/ | puppetserver | puppetserver JVM process |
| /etc/puppet/code/ | puppetserver | puppetserver (code, modules, Hiera) |
| /etc/puppetlabs/puppetserver/ca/ | puppetserver | puppetserver CA subsystem |
| /var/lib/puppet/ssl/ | puppet | agent SSL (certs, keys, CRL) |

The split exists because puppetserver ships its own config layout under /etc/puppet/
while the puppet agent CLI uses /etc/puppetlabs/puppet/. Both are active simultaneously
on the primary since it runs both processes.

## Two puppet.conf files

There are two puppet.conf files with different content. Both are active.

### /etc/puppetlabs/puppet/puppet.conf
Read by the puppet agent CLI only. Contains agent-side settings:

    [main]
    certname = puppet.lab.local
    server = puppet.lab.local
    environment = production

    [agent]
    runinterval = 1h
    reports = store,puppetdb

Do NOT add storeconfigs or puppetdb settings here. Those belong on the server side only.

### /etc/puppet/puppet.conf (puppetserver)
Read by puppetserver only. Contains server-side settings including PuppetDB integration:

    [main]
    certname = puppet.lab.local
    server = puppet.lab.local
    environment = production

    [master]
    storeconfigs = true
    storeconfigs_backend = puppetdb
    reports = store,puppetdb

## Puppetserver conf.d

Puppetserver config is split into multiple files under /etc/puppet/puppetserver/conf.d/:

| File | Purpose |
|---|---|
| ca.conf | CA directory location (cadir) |
| webserver.conf | SSL cert paths, port 8140 |
| puppetserver.conf | JRuby settings, gem path |
| auth.conf | API authorization rules |

## Code and module paths

| Path | Purpose |
|---|---|
| /etc/puppet/code/environments/production/ | Active environment root |
| /etc/puppet/code/environments/production/manifests/ | site.pp |
| /etc/puppet/code/environments/production/modules/ | All modules (roles, profiles, component) |
| /etc/puppet/code/environments/production/data/ | Hiera data |
| /etc/puppet/code/environments/production/hiera.yaml | Hiera config |

## r10k

r10k manages the production environment by pulling from a self-hosted Gitea instance.
Config at /etc/puppetlabs/r10k/r10k.yaml.
r10k runs as root. All git operations on the Puppet primary require sudo.
r10k deploys every 15 minutes via cron. See r10k-workflow.md for the full workflow.

## Hiera hierarchy

Hiera config at /etc/puppet/code/environments/production/hiera.yaml.
Lookup order (highest to lowest precedence):

1. data/nodes/%{trusted.certname}.eyaml  (per-node encrypted)
2. data/nodes/%{trusted.certname}.yaml   (per-node plaintext)
3. data/common.eyaml                     (lab-wide encrypted)
4. data/common.yaml                      (lab-wide plaintext)

eyaml beats yaml at each level. PKCS7 keypair at /etc/puppetlabs/puppet/eyaml/.
Private key backed up to a password manager. See eyaml-secrets.md.

## Role/profile structure

All nodes are assigned exactly one role. Roles compose profiles. Profiles manage resources.

| Role | Assigned to | Profiles |
|---|---|---|
| role::standard_server | plex, cli-docker, puppetdb, super-admin (default) | profile::base |
| role::download_server | rbt | profile::base, profile::vpn, profile::download_client |

profile::base includes profile::node_exporter on all nodes.
