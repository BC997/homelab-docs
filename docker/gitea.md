# Gitea: As-Built Reference

## Host

| Property | Value |
|---|---|
| Host | cli-docker.lab.local |
| IP | 10.0.0.3 |
| Container | gitea |
| Image | gitea/gitea:latest |
| Web UI | http://gitea.lab.local:3001 |
| SSH port | 2222 |
| HTTP port | 3001 |

## Overview

Self-hosted Gitea runs as a Docker container on cli-docker. It hosts all private
homelab repos. r10k on the Puppet primary pulls from Gitea to deploy Puppet code.

## Repositories

| Repo | Purpose |
|---|---|
| homelab-puppet | Puppet code — roles, profiles, Hiera, Puppetfile |
| homelab-docker | Docker compose files for all stacks |
| homelab-proxmox | Proxmox-related configs |
| homelab-haos | Home Assistant OS config |
| homelab-docs | Lab documentation |

## r10k integration

r10k on the Puppet primary is configured to pull from homelab-puppet.
Config at /etc/puppetlabs/r10k/r10k.yaml.

r10k deploys the production branch to:

    /etc/puppet/code/environments/production/

The Gitea access token for r10k is stored as an eyaml encrypted secret in Hiera.

## Git operations on Puppet primary

All git operations require sudo on the Puppet primary since r10k runs as root
and owns the working copy.

    sudo git add <file>
    sudo git commit -m "message"
    sudo git push origin HEAD:production

## Watchtower

Watchtower manages Gitea container updates automatically on cli-docker.

## GitHub mirror

A sanitized public snapshot of homelab-docs is pushed periodically to GitHub.
Real IPs, credentials, and sensitive details are replaced with placeholders.
GitHub is not a live mirror — updated manually when the lab changes significantly.
