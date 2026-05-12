# PuppetDB Integration: As-Built Reference

## Host

| Property | Value |
|---|---|
| Hostname | puppetdb.lab.local |
| IP | 10.0.0.6 |
| Proxmox node | Apricot |
| VM ID | 108 |
| OS | Ubuntu 22.04 |
| Username | puppetdb |
| PuppetDB version | Latest (managed via apt) |

## Overview

PuppetDB stores all agent reports, facts, and exported resources. The Puppet primary
submits reports and queries PuppetDB after every catalog compile. PuppetBoard provides
a web UI on top of PuppetDB.

PuppetBoard: http://10.0.0.6

## Wiring

The integration is configured in two places. Both must be correct or reports will fail.

### Puppetserver side (/etc/puppet/puppet.conf)

The [master] section enables storeconfigs and points reports at PuppetDB:

    [master]
    storeconfigs = true
    storeconfigs_backend = puppetdb
    reports = store,puppetdb

This file is read by puppetserver only. Do NOT add these settings to the agent-side
puppet.conf at /etc/puppetlabs/puppet/puppet.conf — agents only need:

    [agent]
    reports = store,puppetdb

### PuppetDB connection config

    /etc/puppet/puppetserver/conf.d/puppetdb.conf

Points puppetserver at the PuppetDB host and port:

    puppetdb: {
        server_urls = ["https://puppetdb.lab.local:8081"]
        soft_write_failure = false
    }

### PuppetDB terminus

The puppetdb-termini package is installed on the primary. It provides the
storeconfigs_backend integration between puppetserver and PuppetDB.

## Ports

| Port | Protocol | Purpose |
|---|---|---|
| 8081 | HTTPS | puppetserver to PuppetDB (API) |
| 8080 | HTTP | PuppetBoard to PuppetDB (local only) |

## Verify integration

On the primary, after a puppet agent run:

    sudo /opt/puppetlabs/bin/puppet node status puppetdb.lab.local

All six nodes should appear as active in PuppetBoard.

## Gotcha: storeconfigs on agents

Adding storeconfigs to the agent-side puppet.conf (/etc/puppetlabs/puppet/puppet.conf)
will cause agent runs to fail with an error about unknown configuration keys.
storeconfigs is a server-only setting. It belongs only in /etc/puppet/puppet.conf
under [master].
