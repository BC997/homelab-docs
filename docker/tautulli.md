# Tautulli: As-Built Reference
## Overview
Tautulli provides Plex analytics, play history, and notification capabilities.
Runs as a container on cli-docker, managed by Puppet via profile::tautulli.

## Service
| Property | Value |
|---|---|
| Host | cli-docker.lab.local |
| IP | 10.0.0.80 |
| Port | 8181 |
| Config dir | /home/ubuntu/docker/tautulli/config |
| Plex server | 10.0.0.155:32400 |
| Puppet profile | profile::tautulli |

## Puppet
Managed by profile::tautulli. Compose file rendered from template at:
modules/profile/templates/tautulli/docker-compose.yml.erb

Plex token stored as eyaml secret in:
data/nodes/cli-docker.lab.local.eyaml (key: profile::tautulli::plex_token)

## Watchtower
Watchtower on cli-docker handles image updates automatically.
