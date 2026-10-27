# Pi-hole: As Built Reference

## Host

| Property | Value |
|---|---|
| Host | cli-docker, 10.0.0.11 |
| Network mode | host |
| DNS port | 53 |
| Version | v6 |

NPM owns port 80 on cli-docker, so the Pi-hole admin UI runs on a different port.

## API authentication (v6)

v6 replaced the API key with a session token.

1. POST the web password to `/api/auth` to get a session ID
2. Send `X-FTL-SID: <sid>` on every later call

Sessions expire. Re authenticate on a 401.

## Managing DNS records

Add:

    curl -X PUT "http://10.0.0.11:<ADMIN_PORT>/api/config/dns/hosts/<IP>%20<hostname>" -H "X-FTL-SID: <sid>"

Delete:

    curl -X DELETE "http://10.0.0.11:<ADMIN_PORT>/api/config/dns/hosts/<IP>%20<hostname>" -H "X-FTL-SID: <sid>"

List:

    curl "http://10.0.0.11:<ADMIN_PORT>/api/config/dns/hosts" -H "X-FTL-SID: <sid>"

## Key records

| Hostname | IP |
|---|---|
| openvox-cli.lab.local | 10.0.0.10 |
| cli-docker.lab.local | 10.0.0.11 |
| gitea.lab.local | 10.0.0.11 |
| plex-cli.lab.local | 10.0.0.12 |
| ha.<DOMAIN> | 10.0.0.11 |

## Reboot window

cli-docker reboots nightly at 4:30, which takes DNS down for the whole network for a couple of minutes. It runs 30 minutes ahead of the Plex host's 5:00 reboot, so DNS is back before anything else restarts. OpenVox agents don't need Pi-hole to reach the primary, since every node has the primary pinned in `/etc/hosts`.

## Drift risk

Local DNS records are not managed by Puppet. Records for retired hosts linger and new hosts get missed. Records are audited by hand for now.
