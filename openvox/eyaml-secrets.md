# eyaml Secrets: As Built Reference

## Keys

| Property | Value |
|---|---|
| Public key | /etc/puppetlabs/puppet/eyaml/public_key.pkcs7.pem |
| Private key | /etc/puppetlabs/puppet/eyaml/private_key.pkcs7.pem |
| Backup | Password manager, plus an offline copy taken before the OpenVox migration |

The same keypair moved to the OpenVox primary, so no secrets had to be re encrypted. Never commit either key. If the private key is lost, every encrypted value has to be re encrypted with a new pair.

## Encrypted files

| File | Holds |
|---|---|
| data/common.eyaml | Lab wide secrets, such as the admin password hash |
| data/nodes/cli-docker.lab.local.eyaml | Proxmox API password, Pi-hole password, Plex token for Tautulli |

## Encrypting a value

    /opt/puppetlabs/puppet/bin/eyaml encrypt -s 'the secret value'

Paste the `ENC[PKCS7,...]` block into the right .eyaml file:

    profile::some_class::password: >
      ENC[PKCS7,MIIB...]

Then follow the normal workflow in `r10k-workflow.md`.

## Rotating a secret

1. Encrypt the new value
2. Replace the ENC block
3. Commit, push, deploy
4. Noop run, then apply
5. Revoke the old credential only after several successful automated runs
