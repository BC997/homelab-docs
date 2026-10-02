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

r10k on openvox-cli pulls the production branch of homelab-puppet and deploys it to `/etc/puppet/code/environments/production/`. Today it authenticates with an HTTP token stored in eyaml. Moving to an SSH deploy key is on the roadmap.

Day to day edits happen in a normal user clone at `~/repos/homelab-puppet` on openvox-cli, not in the deploy path.

## GitHub snapshot

This GitHub repo is a sanitized copy of homelab-docs. It is updated by hand when the lab changes in a meaningful way. Real IPs, domains, and secrets are replaced before publishing.
