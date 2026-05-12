# Agent Re-bootstrap: As-Built Reference

## Overview

Agent re-bootstrap is required when an agent's SSL state is invalid or out of sync with
the Puppet primary's CA. This happens after a CA rebuild, or when a cert is cleaned and
needs to be reissued.

## Autosign

Autosign is enabled on the primary. Any certname matching *.lab.local is signed
automatically on first agent run. No manual signing needed for lab hosts.

Config: /etc/puppetlabs/puppet/autosign.conf

## Per-host reference

| Certname | Host | IP | Proxmox node |
|---|---|---|---|
| puppet.lab.local | Butternut VM 106 | 10.0.0.2 | Butternut |
| cli-docker.lab.local | Dessert VM 110 | 10.0.0.3 | Dessert |
| plex.lab.local | Echo VM 101 | 10.0.0.4 | Echo |
| rbt.lab.local | Echo VM 107 | 10.0.0.5 | Echo |
| super-admin.lab.local | Echo VM 104 | Echo | Echo |
| puppetdb.lab.local | Apricot VM 108 | 10.0.0.6 | Apricot |

## Re-bootstrap procedure

Run these steps on each agent host.

### Step 1: Clean the cert on the primary

On puppet.lab.local:

    sudo /usr/bin/puppetserver ca clean --certname <certname>

### Step 2: Wipe SSL state on the agent

On the agent host:

    sudo rm -rf /var/lib/puppet/ssl/*

### Step 3: Run the agent

On the agent host:

    sudo /opt/puppetlabs/bin/puppet agent -t

The agent will generate a new CSR, submit it to the primary, autosign will approve it,
and the agent will receive a new cert signed by the active CA. A full catalog run
will follow immediately.

### Step 4: Verify on the primary

On puppet.lab.local:

    sudo /usr/bin/puppetserver ca list --all

Confirm the certname appears in the signed list. Also verify in PuppetBoard that
the node is reporting.

## Re-bootstrap order after a CA rebuild

If re-bootstrapping the full fleet after a CA rebuild, do it in this order:

1. puppet.lab.local (primary itself — run agent locally, skip Step 1)
2. cli-docker.lab.local
3. plex.lab.local
4. rbt.lab.local
5. super-admin.lab.local
6. puppetdb.lab.local

PuppetDB must be last because other nodes depend on it for report storage.
