# Puppet CA: As-Built Reference

## Active CA (CA-C)

CA-C was established during a CA reconciliation that retired two prior CAs (CA-A and CA-B).
CA-A files are preserved at /etc/puppet/puppetserver/ca.CA-A.preserved/ for reference only.

| Property | Value |
|---|---|
| Fingerprint (SHA-256) | <CA_FINGERPRINT> |
| Subject | CN = Puppet CA: puppet.lab.local |
| Valid | 2026-05-09 to 2031-05-09 |
| CA directory | /etc/puppetlabs/puppetserver/ca/ |
| CA public cert | /etc/puppetlabs/puppetserver/ca/ca_crt.pem |
| CA private key | /etc/puppetlabs/puppetserver/ca/ca_key.pem |
| CRL | /etc/puppetlabs/puppetserver/ca/ca_crl.pem |
| Signed certs | /etc/puppetlabs/puppetserver/ca/signed/ |
| Inventory | /etc/puppetlabs/puppetserver/ca/inventory.txt |
| Trust anchor (agents + puppetserver) | /var/lib/puppet/ssl/certs/ca.pem |
| cadir config | /etc/puppet/puppetserver/conf.d/ca.conf |

## Fleet cert inventory

All six agents are signed by CA-C:

| Serial | Certname | Host |
|---|---|---|
| s2 | puppet.lab.local | Butternut 10.0.0.2 |
| s3 | cli-docker.lab.local | Dessert 10.0.0.3 |
| s4 | plex.lab.local | Echo 10.0.0.4 |
| s5 | qby.lab.local | Echo 10.0.0.5 |
| s6 | super-admin.lab.local | Echo |
| s7 | puppetdb.lab.local | Apricot 10.0.0.6 |

## Verify CA health

    # Confirm active CA fingerprint matches expected value
    sudo openssl x509 -in /etc/puppetlabs/puppetserver/ca/ca_crt.pem -noout -fingerprint -sha256

    # Confirm trust anchor on disk matches
    sudo openssl x509 -in /var/lib/puppet/ssl/certs/ca.pem -noout -fingerprint -sha256

    # List all signed/pending/revoked certs
    sudo /usr/bin/puppetserver ca list --all

If the two fingerprints do not match, or puppetserver ca list returns SSL errors,
the trust anchor is out of sync with the active CA. Sync it by copying
ca_crt.pem to /var/lib/puppet/ssl/certs/ca.pem and restarting puppetserver.

## Cleaning a stale cert

puppet cert does not exist in Puppet 8. Use:

    sudo /usr/bin/puppetserver ca clean --certname <certname>

Then re-bootstrap the agent. See agent-rebootstrap.md.
