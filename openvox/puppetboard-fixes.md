# PuppetBoard and Server Fixes

These were applied on openvox-cli after the migration. They sit outside Puppet managed paths, so they are listed here until they are codified.

| File | Change | Puppet managed |
|---|---|---|
| routes.yaml | Facts and catalog terminus set to OpenVoxDB | No |
| auth.conf | Allow metrics endpoint access | No |
| PuppetBoard index.py | Graceful fallback when metrics return 403 | No |
| PuppetBoard dailychart.py | Day boundary calculated in local time instead of UTC | No |

## Risk

A package upgrade or a rebuild of the primary will silently undo these fixes. Symptoms would be missing facts or catalogs in OpenVoxDB, an error on the PuppetBoard overview, or daily run counts that split at the wrong hour.

## Plan

Move each fix into a profile on the primary so it is enforced on every run. Until then, the scripts in the internal homelab-puppet repo (`docs/puppetboard-fixes.sh` and `docs/openvox-master-setup.sh`) reapply them.
