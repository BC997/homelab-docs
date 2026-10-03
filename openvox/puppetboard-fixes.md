# PuppetBoard and Server Fixes

These fixes were applied by hand on openvox-cli after the OpenVox migration. Since October 2026, `profile::openvox_primary` enforces them on every agent run.

| File | Change | Enforced by |
|---|---|---|
| routes.yaml | Facts and catalog terminus set to OpenVoxDB | Managed file |
| autosign.conf | Explicit allowlist of certnames, from Hiera | Managed file |
| OpenVoxDB auth.conf | Allow metrics endpoint access | Helper script |
| PuppetBoard index.py | Graceful fallback when metrics return 403 | Helper script |
| PuppetBoard dailychart.py | Day boundary calculated in local time instead of UTC | Helper script |

## How it works

A helper script, installed by the profile, has a `check` mode and an `apply` mode for each fix. Puppet runs `apply` only when `check` reports that the fix is missing.

* A clean run does nothing.
* If a package upgrade undoes a fix, the next run puts it back and restarts only the service that needs it.
* If an upgrade changes the code a patch expects, `apply` fails instead of guessing. The run shows as failed in PuppetBoard, which is the signal to look at the new code.

This was tested by undoing the dailychart fix by hand. The next agent run reported it as a corrective change, restored it, and restarted PuppetBoard.

## Not managed

`server-code-dir` in puppetserver.conf stays manual. If it is wrong, the server cannot load any code, including the profile that would repair it. It is set by the bootstrap script on a fresh primary.

## Why the autosign list moved to Hiera

Once Puppet manages autosign.conf, a hand edit on the primary gets overwritten on the next run. New nodes are added to `profile::openvox_primary::autosign_certnames` in the primary's node data instead.
