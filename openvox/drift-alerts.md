# Drift and Failure Alerts: As Built Reference

## What it does

Every failed agent run, and every change OpenVox makes outside a planned deploy, becomes a phone notification through Home Assistant. Each alert carries the full report log, the same lines PuppetBoard shows. A VM that changes without a planned change is often the first sign of trouble, so this closes the gap between "PuppetBoard shows it" and "someone actually saw it".

| Property | Value |
|---|---|
| Runs on | openvox-cli (10.0.0.10) |
| Profile | `profile::puppet_report_alert` |
| Check interval | Every 5 minutes (systemd timer) |
| Source of truth | OpenVoxDB reports API on localhost |
| Destination | Home Assistant webhook, LAN only |
| Logs | `sudo journalctl -u puppet-report-alert.service` |

## Telling drift apart from planned change

OpenVoxDB has a `corrective_change` field on each report, but the open source server leaves it null, so it cannot be used. Timing is used instead:

* A **failed** report always alerts.
* A **changed** report alerts only when it lands more than 45 minutes after the last r10k deploy. The deploy time comes from the mtime of `.r10k-deploy.json` in the deployed environment.
* A changed report inside that window is treated as the rollout of a planned change and is skipped.

Agents run every 30 minutes, so 45 minutes covers one full round of agent runs after a deploy.

## Flow

    agent run on any VM
      -> report stored in OpenVoxDB
      -> checker on openvox-cli (every 5 min) reads reports newer than the last one it saw
      -> failed, or changed more than 45 min after the last deploy?
           no  -> skipped, logged as planned
           yes -> POST to the Home Assistant webhook
                   -> phone notification with the full report log
                   -> entry in a 7 day Alert history list

## Payload

The checker sends one JSON object per report:

| Field | Contents |
|---|---|
| host | Short certname |
| status | failed or changed |
| resources | Resources changed in that run, from the events API |
| version | Catalog configuration version |
| time | When OpenVoxDB received the report |
| log | Report log lines, formatted like PuppetBoard and capped at about 3,000 characters |

## Design notes

* **The webhook URL is a secret.** It lives in per node eyaml, is written to a root only environment file (`/etc/default/puppet-report-alert`, mode 0600) with `Sensitive` and `show_diff => false`, and never appears in the unit file or in reports. Its run shows as `changed [redacted] to [redacted]`.
* **No missed reports.** The checker keeps the newest report time it handled in `/var/lib/puppet-report-alert/last_seen`. If the webhook call fails, it stops and leaves that marker alone, so the next run retries the same report.
* **No flood on first run.** With no marker yet, the first run writes the current time and exits instead of alerting on old reports.
* **Python standard library only.** The checker needs nothing beyond python3, which the profile ensures.
* **Clean removal.** Setting `profile::puppet_report_alert::ensure: absent` in Hiera stops the timer and removes the script, units, and environment file.

## Testing

1. A hand run of the service on the primary right after deploying logged `change within 45 min of a deploy, treated as intentional`, which proved the deploy window works.
2. A test POST to the webhook produced the phone notification and the Alert history entry.
3. For a real drift test, wait more than 45 minutes after a deploy, change a managed resource by hand on a VM, and let the next agent run put it back.

## Files

| File | Purpose |
|---|---|
| modules/profile/manifests/puppet_report_alert.pp | Script, environment file, systemd service and timer |
| modules/profile/files/puppet_report_alert/puppet-report-alert | The checker |
| data/nodes/openvox-cli.lab.local.eyaml | Encrypted webhook URL (not published) |
