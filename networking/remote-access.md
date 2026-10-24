# Remote Access: As Built Reference

## Overview

Nginx Proxy Manager (NPM) on cli-docker is the main entry point for external traffic. WAN 443 goes to NPM, which routes by hostname. Port 80 is closed.

## Domain

| Property | Value |
|---|---|
| Domain | <DOMAIN> |
| Registrar | Cloudflare |
| DNS | Cloudflare, DNS only (no Cloudflare proxy) |
| API token | Scoped to Edit zone DNS for this one zone, kept in a password manager |

## Dynamic DNS

A cloudflare-ddns container on cli-docker updates the A records for `ha.<DOMAIN>` and `plex.<DOMAIN>` every five minutes. DuckDNS was used before and is fully removed.

## NPM

| Property | Value |
|---|---|
| Host | cli-docker, 10.0.0.11 |
| Admin UI | 10.0.0.11:81 |
| Certificates | Let's Encrypt using the Cloudflare DNS challenge |

| Hostname | Backend |
|---|---|
| ha.<DOMAIN> | 10.0.0.13:8123 over HTTP |
| plex.<DOMAIN> | 10.0.0.12:32400 over HTTP |

NPM terminates TLS. Backend traffic stays on the LAN in plain HTTP.

## Plex

Moving Plex fully behind NPM did not work, so Plex is still reached through its own port forward to plex-cli. Geo blocking in UniFi was tried for that port and dropped because it caused too much friction.

## NAT loopback

The gateway does not do NAT hairpin, so Pi-hole answers for the external hostnames from inside the LAN:

| Hostname | Resolves to |
|---|---|
| ha.<DOMAIN> | NPM (10.0.0.11) |

## Home Assistant

HAOS accepts proxied traffic from NPM with IP banning on failed logins. The HTTP and trusted proxy settings now live in the Home Assistant UI under Settings > System > Network instead of configuration.yaml. See `haos/architecture.md`.
