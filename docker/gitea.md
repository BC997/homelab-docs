# Gitea: As Built Reference

## Host

| Property | Value |
|---|---|
| Host | cli-docker, 10.0.0.11 |
| Web UI | http://gitea.lab.local:3001 |
| SSH port | 2222 |

## Repositories

| Repo | Purpose |
|---|---|
| homelab-puppet | OpenVox code: roles, profiles, Hiera, Puppetfile, setup scripts |
| homelab-docker | Compose files |
| homelab-proxmox | Proxmox configs |
| homelab-haos | Home Assistant config |
| homelab-docs | Lab documentation, source for this GitHub snapshot |

A separate `homelab-ansible` repo is planned so Ansible stays outside the r10k deploy path.

## r10k integration

r10k on openvox-cli pulls the production branch of homelab-puppet over SSH (port 2222) and deploys it to `/etc/puppet/code/environments/production/`. It uses a read only deploy key that only reaches that one repo.

Day to day edits happen in a normal user clone at `~/repos/homelab-puppet` on openvox-cli, not in the deploy path. That clone pushes over SSH with its own key.

The Gitea account has no access tokens. The last one was retired in October 2026, after an audit found every place that used it and moved each one to SSH first.

## GitHub snapshot

This GitHub repo is a sanitized copy of homelab-docs. It is updated by hand when the lab changes in a meaningful way. Real IPs, domains, and secrets are replaced before publishing.
