# Seerr: As-Built Reference
## Overview
Seerr is a media request management and discovery tool for Plex.
It is the official successor to Overseerr and Jellyseerr, which merged into a single project in February 2026.
Runs as a container on cli-docker, managed by Puppet via profile::seerr.

## Service
| Property | Value |
|---|---|
| Host | cli-docker.lab.local |
| IP | 10.0.0.80 |
| Port | 5055 |
| Config dir | /home/ubuntu/docker/seerr/config |
| Container name | seerr |
| Image | ghcr.io/seerr-team/seerr:latest |
| Puppet profile | profile::seerr |
| Access | LAN only (http://10.0.0.80:5055) |

## Integrations
| Service | Host | Port |
|---|---|---|
| Plex | 10.0.0.155 | 32400 |
| Radarr | 10.0.0.80 | 7878 |
| Sonarr | 10.0.0.80 | 8082 |

## Puppet
Managed by profile::seerr. Compose file rendered from template at:
modules/profile/templates/seerr/docker-compose.yml.erb
Config directory ownership is 1000:1000 (required by Seerr image).

## Notes
Migrated from Overseerr on May 25 2026. Config was automatically migrated by Seerr on first startup.
Intentionally LAN-only — no reverse proxy or external DNS entry.

## Watchtower
Watchtower on cli-docker handles image updates automatically.
