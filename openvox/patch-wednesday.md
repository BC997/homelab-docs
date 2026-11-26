# Patch Wednesday

A weekly patch window for the OpenVox managed VMs: package updates on Wednesday at 01:00, then each VM reboots in its own slot only if a reboot is actually pending. Managed by OpenVox in the homelab code repo. Added November 2026.

## Why

- The OpenVox primary and the Docker host had no scheduled reboots, so kernel and library updates sat installed but unused until someone rebooted by hand. In November 2026 both were a kernel behind what was installed.
- The Plex host rebooted every day whether it needed to or not.
- Sunday is already busy with VM backups, the weekly container update, and Home Assistant's own updates. Wednesday keeps patching away from all of it.

## Schedule (Wednesday, Pacific time)

| Time | What | Where it is set |
|---|---|---|
| 01:00 | Weekly apt (`apt-get update && apt-get -y upgrade`) on every managed VM | `cron weekly-apt-upgrade` in profile::base, `weekday => '3'` |
| 03:30 | openvox-cli reboots, only if a reboot is pending | node Hiera |
| 03:45 | plex-cli reboots, only if a reboot is pending | node Hiera |
| 04:15 | cli-docker reboots, only if a reboot is pending. Last, because it carries DNS and every container | node Hiera |
| about 5 to 15 min after cli-docker boots | container-check confirms every auto restart container is up | profile::container_check |

- On a quiet week nothing reboots.
- Daily Ubuntu security updates (unattended-upgrades) keep running every day.
- The Plex host's daily reboot is gone (`profile::base::manage_nightly_reboot: false`). The nightly reboot code in profile::base stays as an option, but no VM uses it.
- Sunday keeps the VM backups and the weekly container update.

## How a patch reboot decides

`patch-reboot.timer` fires once a week in the VM's slot and runs `/usr/local/sbin/patch-reboot` as root. The script:

1. Exits with "No reboot pending, nothing to do" if `/var/run/reboot-required` does not exist.
2. Tests the dpkg locks, `/var/lib/dpkg/lock-frontend` and `/var/lib/dpkg/lock`, with a non blocking fcntl lock and releases them at once. If either is held (apt, dpkg or unattended-upgrades is working), it rechecks every 5 minutes for up to 10 minutes, then gives up for the week. The 10 minute cap keeps each VM inside its 15 minute slot.
3. Skips if the VM has been up less than 30 minutes (`min_uptime_minutes`).
4. Otherwise logs the packages waiting on a reboot and runs `systemctl reboot`, a clean shutdown.

A skipped week is not lost: a separate reboot pending check alerts Home Assistant once the flag is 48 hours old.

Why the lock test and not a process list: unattended-upgrades.service keeps `unattended-upgrade-shutdown --wait-for-signal` running from boot, and its process name is truncated to `unattended-upgr`. A check by process name would have skipped every reboot.

The timer has `Persistent=false`, so a VM that was off during its slot does not reboot when it starts.

## profile::patch_reboot

Included from profile::base.

| Parameter | Default | Notes |
|---|---|---|
| ensure | absent | A node opts in with `profile::patch_reboot::ensure: 'present'` |
| weekday | Wed | |
| time | 03:30 | `HH:MM`, set per node with `profile::patch_reboot::time` |
| min_uptime_minutes | 30 | |

Files on each VM: `/usr/local/sbin/patch-reboot`, `/etc/systemd/system/patch-reboot.service` (oneshot, `TimeoutStartSec=20min`), `/etc/systemd/system/patch-reboot.timer`. With `ensure: absent` the timer is disabled and the files are removed, and a node that never had them shows no change.

Commands:

```
sudo patch-reboot --dry-run            # check once, print the decision, never wait or reboot (needs root)
systemctl list-timers patch-reboot.timer
journalctl -u patch-reboot.service
```

## profile::container_check (Docker host)

Included for cli-docker in `manifests/site.pp`. After every boot, planned or not, it checks that every container with restart policy `unless-stopped` or `always` came back, and alerts Home Assistant once if any did not. It never starts or changes a container.

| Parameter | Default | Notes |
|---|---|---|
| ha_webhook_url | none | `Variant[String, Sensitive[String]]`, from encrypted node Hiera |
| expected_count | 19 | Fewer auto restart containers than this also alerts |
| ignore | grafana, unifi-poller, loki, influxdb, promtail | Disabled containers (restart policy `no`) |
| wait_minutes | 10 | Rechecks every 60 s for this long before alerting |

Files: `/usr/local/bin/container-check`, `/etc/default/container-check` (root only, holds the webhook URL, never shown in Puppet reports), `container-check.service`, `container-check.timer` (`OnBootSec=5min`, once per boot).

Commands:

```
sudo container-check --status          # check now and print, send nothing
sudo container-check --test            # send a TEST alert to Home Assistant
journalctl -u container-check.service
```

First run: "19 of 19 auto restart containers running (expected 19, ignoring 5)".

Why it exists: after a switch reboot fenced a Proxmox node and hard stopped the Docker host, Docker restarted every container but Prometheus, which stayed down for about 20 hours before anyone noticed.

## Prometheus reload rule

Reload Prometheus after a config change with:

```
sudo docker exec prometheus kill -HUP 1
```

Never use `docker kill -s HUP prometheus`. Docker treats any `docker kill`, whatever the signal, as a manual stop and sets the container's `HasBeenManuallyStopped` flag. A container with that flag and restart policy `unless-stopped` is not started again after a reboot or a Docker restart. This was confirmed in the lab: the flag was true after a HUP sent with `docker kill`, and `docker restart prometheus` cleared it. `docker exec prometheus kill -HUP 1` sends the signal inside the container, so Docker never sees a kill. It was tested: the config reloaded, the flag stayed false, and nothing restarted. prometheus.yml is a single file bind mount, so edit it in place (never `sed -i`), then run `promtool check config`, then the HUP.

## Home Assistant

| Automation | Now |
|---|---|
| Container check alert (new) | Receives container-check reports on a LAN only webhook. Phone notice plus an open Alert history item |
| Plex stream problem | Skips Wednesday 01:00 to 01:30 (weekly apt), Wednesday 03:45 to 04:00 (Plex host slot) and the Sunday backup window. The old daily 05:00 window is gone |
| Lab host unreachable | No skip windows at all. A host has to stay unreachable for 3 minutes, a patch reboot finishes in under 2, and a VM that doesn't come back still alerts |
| Watchtower report | Skips only the Watchtower startup message on Wednesday 04:15 to 04:45, so a Docker host patch reboot does not page. The weekly run report, and startup messages at any other time, still alert |

## Checks after a patch night

```
uptime                                  # on each VM: did it reboot in its slot?
journalctl -u patch-reboot.service      # the decision and the packages
journalctl -u container-check.service   # on cli-docker, after its reboot
```

Then Home Assistant Alert history: no host down alerts, and no container check alert.

## Rollback

1. Revert the Patch Wednesday commits in the code repo and push.
2. `sudo /opt/puppetlabs/puppet/bin/r10k deploy environment production -pv`.
3. Noop, then a real agent run on each VM. Reverting turns `ensure` back to the default `absent`, which disables the timers and removes the files.
4. Reverting also restores the Sunday 01:00 apt cron and the Plex host's daily reboot.
5. Put the Home Assistant automations back from the backup of their old conditions.

To stop patch reboots on one VM only, set `profile::patch_reboot::ensure: 'absent'` in its node Hiera.

## Lessons

- `puppet parser validate` only checks syntax. container_check.pp first used `${wait_minutes * 60 + 300}` in a heredoc, which needs `$wait_minutes` inside an expression. It passed validate and failed only when the Docker host's catalog compiled. Compute values into a variable first, then interpolate the variable.
- A noop is a preview, not a gate: once the code is deployed, each agent's next scheduled run applies it.
