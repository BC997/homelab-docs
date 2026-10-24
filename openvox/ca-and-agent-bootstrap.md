# CA, Autosign, and Agent Bootstrap

## CA

The CA lives on the primary, openvox-cli. Every managed node received a new cert from it during the migration. The CA fingerprint is kept privately and never committed here.

## Autosign

Autosign uses an explicit allowlist of certnames instead of a `*.lab.local` wildcard. A host that is not on the list submits a CSR and waits unsigned. That is intended. Nothing joins the fleet without being added on purpose.

The list lives in Hiera (`profile::openvox_primary::autosign_certnames`), and Puppet writes autosign.conf from it. Editing autosign.conf by hand on the primary does not stick.

## Onboarding a new VM

1. Create the VM with its NIC on vmbr0 and install Ubuntu Server 24.04. There is no template right now, and rebuilding one is on the roadmap.
2. Set a DHCP reservation in UniFi
3. Set the hostname so the certname is exact
4. Add a Pi-hole Local DNS record, plus a line in the workstation's hosts file (see `networking/workstation-dns.md`)
5. Add the certname to the autosign list in the primary's Hiera data, then commit, deploy, and run the agent on the primary
6. If the VM needs more than the base profile, add a node block in site.pp through the normal workflow
7. Run the agent on the new VM

       sudo /opt/puppetlabs/bin/puppet agent -t

8. Confirm the node is reporting in PuppetBoard

Skipping step 6 does not fail loudly. The node falls through to `node default` and only gets the base profile.

## Re bootstrapping an agent

On the primary:

    sudo /opt/puppetlabs/bin/puppetserver ca clean --certname <certname>

On the agent, clear SSL state and run again:

    sudo rm -rf /var/lib/puppet/ssl/*

    sudo /opt/puppetlabs/bin/puppet agent -t

## Gotcha

`storeconfigs` belongs on the server only. Adding it to an agent's puppet.conf breaks agent runs.
