
# Pi-hole: As-Built Reference

## Host

| Property | Value |
|---|---|
| Host | cli-docker.lab.local |
| IP | 10.0.0.80 |
| Container | pihole |
| Image | pihole/pihole:latest |
| Web UI | http://10.0.0.80/admin |
| DNS port | 53 |
| Version | v6 |

## Overview

Pi-hole v6 runs on cli-docker as a Docker container with network_mode: host.
It provides local DNS resolution for all lab.local hostnames and ad blocking
for the entire LAN. All Proxmox nodes, VMs, and services have DNS records here.

## API authentication (v6 change)

Pi-hole v6 changed authentication from an API key to a session token.
The old X-API-Key header no longer works.

Authentication flow:
1. POST to /api/auth with the web password to get a session ID (sid)
2. Use X-FTL-SID: <sid> header on all subsequent API calls

Session tokens expire — re-authenticate if API calls start returning 401.

## DNS record management via API

### Add a DNS record
    curl -X PUT http://10.0.0.80/api/config/dns/hosts/<IP>%20<hostname> \
      -H "X-FTL-SID: <sid>"

Example — add puppet.lab.local:
    curl -X PUT http://10.0.0.80/api/config/dns/hosts/10.0.0.246%20puppet.lab.local \
      -H "X-FTL-SID: <sid>"

### Delete a DNS record
    curl -X DELETE http://10.0.0.80/api/config/dns/hosts/<IP>%20<hostname> \
      -H "X-FTL-SID: <sid>"

### List all DNS records
    curl http://10.0.0.80/api/config/dns/hosts \
      -H "X-FTL-SID: <sid>"

## Local DNS records

All lab hosts are registered in Pi-hole. Key records:

| Hostname | IP |
|---|---|
| puppet.lab.local | 10.0.0.246 |
| puppetdb.lab.local | 10.0.0.247 |
| cli-docker.lab.local | 10.0.0.80 |
| plex.lab.local | 10.0.0.155 |
| qby.lab.local | 10.0.0.147 |
| gitea.lab.local | 10.0.0.80 |
| ha.example.com | 10.0.0.80 |
| plex.example.com | 10.0.0.80 |

ha.example.com and plex.example.com resolve to NPM (10.0.0.80) internally
for NAT loopback — see networking/remote-access.md.

## Docker config note

Pi-hole runs with network_mode: host — it binds directly to the host's
network stack on port 53. This means it cannot share port 53 with any
other service on cli-docker.

## Watchtower

Watchtower manages Pi-hole container updates automatically on cli-docker.
