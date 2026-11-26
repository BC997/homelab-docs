# plex-watch: remote Plex buffering alerts

Added Nov 2026. Managed by OpenVox as `profile::plex_watch` on plex-cli (10.0.0.185).

## What it does

plex-watch tells Home Assistant when a remote viewer's Plex stream buffers or fails, and says whose side the problem is on.

- Listens to Plex's local websocket for play session state changes, so short buffers are not missed between polls.
- Ignores what a person does on purpose: a pause, the first 15 seconds after a play start, a resume, a seek (each seek restarts the window, so a run of 10 second skips never alerts), and a quality change.
- Reports remote (WAN) viewers only. LAN viewers are skipped.
- On a real buffer of 2 seconds or more, reads the session details and posts one report to the HA automation "Plex Stream Problem" (`automation.plex_stream_problem`, LAN only webhook). A buffer still going at 10 seconds is reported while it lasts. One alert per session every 2 minutes; the next one says how many it skipped.
- Follows the Plex log live, since Plex keeps only about 4 hours of logs on disk. A stop right after a file or transcoder error is reported as an error.

## Likely cause in each alert

| Cause | When |
|---|---|
| plex-cli can not keep up | Transcoding, speed under 1.0x and not throttled |
| Viewer connection | Plex keeping up (direct play, or a transcode that is throttled or above 1.0x) but the remote player buffers |
| Plex relay | The session is relayed, so remote access broke and Plex fell back to its capped relay |
| Media file or NAS | File read or NFS errors in the Plex log |
| Playback error | A stop right after a transcoder error |

Each alert also shows direct play or transcode (and why: "quality set lower than the file" or "player likely can't play"), the file vs what is being sent, reserved bandwidth (Plex budgets for the file's peaks, not its average), and plex-cli CPU.

## Files

| Item | Location |
|---|---|
| Profile | `modules/profile/manifests/plex_watch.pp` |
| Script | `modules/profile/files/plex_watch/plex-watch` (Python) |
| Role | `include profile::plex_watch` in `modules/role/manifests/media_server.pp` |
| Hiera | `data/nodes/plex-cli.lab.local.eyaml`, key `profile::plex_watch::ha_webhook_url` (encrypted) |
| On plex-cli | `/usr/local/sbin/plex-watch`, `/etc/default/plex-watch` (0600), `plex-watch.service` |

The Plex token is never stored in OpenVox. The service runs as the `plex` user and reads `PlexOnlineToken` from Plex's own Preferences.xml at run time.

## Commands (plex-cli)

```
sudo systemctl status plex-watch --no-pager
sudo journalctl -u plex-watch | cat
sudo /usr/local/sbin/plex-watch --check
sudo /usr/local/sbin/plex-watch --test
```

- `--check` prints live sessions as plex-watch sees them (location, direct play or transcode, reason, speed, reserved bandwidth) and plex-cli CPU. It never prints the token.
- `--test` sends a TEST alert to HA.
- The journal logs each play, pause and stop, every "buffering ignored (play start, resume, seek or quality change)", and every report sent with HA's answer.

## Tuning

Defaults, set as environment values the script reads: START_GRACE 15 s, SEEK_GRACE 15 s, SEEK_JUMP_MS 4000, MIN_BUFFER 2 s, LONG_BUFFER 10 s, COOLDOWN 120 s.

## HA side

`automation.plex_stream_problem` skips planned windows: Wednesday 01:00 to 01:30 (weekly apt), Wednesday 03:45 to 04:00 (plex-cli Patch Wednesday reboot, see `openvox/patch-wednesday.md`) and Sunday 02:15 to 03:15 (backup). The old daily 05:00 window is gone. It sends a phone notice and an open Alert history item.

## Background

Built after a remote viewer could not finish a 4K HEVC HDR movie (7.9 Mbps average, about 19 Mbps peaks). Testing showed plex-cli transcodes 4K HDR on the CPU at 2 to 3x (no GPU in the VM), so the server was not the problem: the viewer's own connection could not hold the peaks on direct play.
