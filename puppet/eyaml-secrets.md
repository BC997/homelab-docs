# eyaml Secrets: As-Built Reference

## Overview

hiera-eyaml is used to encrypt secrets in Hiera data. Encrypted values live in .eyaml
files alongside plaintext .yaml files. eyaml takes precedence over yaml at every level
of the Hiera hierarchy.

## Key locations

| Property | Value |
|---|---|
| PKCS7 public key | /etc/puppetlabs/puppet/eyaml/public_key.pkcs7.pem |
| PKCS7 private key | /etc/puppetlabs/puppet/eyaml/private_key.pkcs7.pem |
| Key permissions | 0600 puppet:puppet |
| Private key backup | Password manager |

Do not commit either key to git. The private key is required to decrypt secrets at
catalog compile time. If it is lost, all encrypted values must be re-encrypted with
a new keypair.

## Installation

hiera-eyaml is installed in two Ruby environments on the primary — both are required:

    /opt/puppetlabs/bin/gem install hiera-eyaml        (agent Ruby)
    /opt/puppetlabs/server/bin/gem install hiera-eyaml (puppetserver JRuby)

## Encrypted files

| File | Scope | Contents |
|---|---|---|
| data/common.eyaml | Lab-wide | Gitea access token |
| data/nodes/cli-docker.lab.local.eyaml | cli-docker only | Proxmox password, Pi-hole password |

Plaintext versions of these secrets have been removed from .yaml files.

## Encrypting a new secret

On the Puppet primary:

    eyaml encrypt -s 'the secret value'

Copy the ENC[PKCS7,...] block into the appropriate .eyaml file under the Hiera data
directory. Follow the standard r10k workflow to commit and deploy. See r10k-workflow.md.

## .eyaml file format

eyaml files use standard YAML syntax. Encrypted values use the ENC[] wrapper:

    profile::some_class::password: >
      ENC[PKCS7,MIIB...]

The > is a YAML literal block scalar — required for multi-line ENC blocks.

## Hiera precedence

eyaml files are configured above their yaml counterparts in hiera.yaml:

    - name: Per-node eyaml
      path: data/nodes/%{trusted.certname}.eyaml
    - name: Per-node yaml
      path: data/nodes/%{trusted.certname}.yaml
    - name: Common eyaml
      path: data/common.eyaml
    - name: Common yaml
      path: data/common.yaml

This means an encrypted value in common.eyaml will always win over a plaintext
value for the same key in common.yaml.

## Rotating a secret

1. Encrypt the new value with eyaml encrypt
2. Replace the ENC[] block in the .eyaml file
3. Commit, push, r10k deploy
4. Verify with a noop run before applying
