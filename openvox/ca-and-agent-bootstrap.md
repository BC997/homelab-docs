# CA, Autosign, and Agent Bootstrap

## CA

The CA lives on the primary, openvox-cli. Every managed node received a new cert from it during the migration. The CA fingerprint is kept privately and never committed here.

## Autosign

Autosign uses an explicit allowlist of certnames instead of a `*.lab.local` wildcard. A host that is not on the list submits a CSR and waits unsigned. That is intended. Nothing joins the fleet without being added on purpose.

## Onboarding a new VM

1. Set a DHCP reservation in UniFi
2. Clone the Ubuntu Server template in Proxmox and attach it to vmbr0
3. Set the hostname so the certname is exact
4. Add a Pi-hole Local DNS record for the new host (not Puppet managed, easy to forget)
5. Add the certname to the autosign allowlist on the primary
6. If the VM needs more than the base profile, add a node block in site.pp through the normal workflow
7. Run the agent on the new VM

       sudo /opt/puppetlabs/bin/puppet agent -t

8. Confirm the node is reporting in PuppetBoard

Skipping step 6 does not fail loudly. The node falls through to `node default` and only gets the base profile.

## Re bootstrapping an agent

On the primary:

    sudo puppetserver ca clean --certname <certname>

On the agent, clear SSL state and run again:

    sudo rm -rf /var/lib/puppet/ssl/*

    sudo /opt/puppetlabs/bin/puppet agent -t

## Gotcha

`storeconfigs` belongs on the server only. Adding it to an agent's puppet.conf breaks agent runs.
