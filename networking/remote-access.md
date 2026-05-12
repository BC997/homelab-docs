# Remote Access: As-Built Reference

## Overview

External access to lab services is handled by Nginx Proxy Manager (NPM) on cli-docker.
All traffic enters on WAN:443 and is routed by NPM based on hostname.
No services are exposed directly — everything goes through NPM.

## Domain

| Property | Value |
|---|---|
| Domain | <DOMAIN> |
| Registrar | Cloudflare |
| DNS management | Cloudflare DNS |
| Cloudflare API token | Scoped to Edit zone DNS, stored in password manager |

## DNS records (Cloudflare)

Both records are A records pointing to the home WAN IP. DNS only (no Cloudflare proxy).

| Record | Type | Target | Proxy |
|---|---|---|---|
| ha.<DOMAIN> | A | <WAN_IP> | DNS only |
| plex.<DOMAIN> | A | <WAN_IP> | DNS only |

## WAN port forwarding

| WAN port | Protocol | Destination |
|---|---|---|
| 443 | HTTPS | NPM (10.0.0.3:443) |

Port 80 is closed. Port 32400 (Plex relay) is closed.

## Nginx Proxy Manager

| Property | Value |
|---|---|
| Host | cli-docker.lab.local |
| IP | 10.0.0.3 |
| Admin UI | 10.0.0.3:81 |
| SSL | Let's Encrypt via Cloudflare DNS challenge |

## Proxy hosts

| Hostname | Backend | Protocol | Notes |
|---|---|---|---|
| ha.<DOMAIN> | 10.0.0.8:8123 | HTTP | HAOS, no SSL internally |
| plex.<DOMAIN> | 10.0.0.4:32400 | HTTP | Plex, no SSL internally |

NPM handles SSL termination. Backend connections from NPM to services are plain HTTP.

## Let's Encrypt

Certificates are issued via Cloudflare DNS challenge using a scoped API token.

## Pi-hole NAT loopback

Internal clients resolve ha.<DOMAIN> and plex.<DOMAIN> to NPM (10.0.0.3)
rather than the WAN IP. This is configured in Pi-hole local DNS:

| Hostname | Resolves to |
|---|---|
| ha.<DOMAIN> | 10.0.0.3 |
| plex.<DOMAIN> | 10.0.0.3 |

Without this, internal clients attempting to reach external hostnames would hit
the WAN IP and fail due to NAT hairpin not being configured on the gateway.

## HAOS HTTP config

HAOS is configured to accept proxied traffic from NPM. Relevant configuration.yaml settings:

    http:
      ip_ban_enabled: true
      login_attempts_threshold: 2
      use_x_forwarded_for: true
      trusted_proxies:
        - 10.0.0.1
        - 10.0.0.0/24

No SSL is configured inside HAOS itself. NPM handles all SSL.

## Plex external access

Plex Remote Access (plex.tv relay) is disabled. External access is via
https://plex.<DOMAIN> only. End users must manually add plex.<DOMAIN>
as a custom server URL in their Plex client.
