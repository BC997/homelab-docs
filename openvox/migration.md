# Open Source Puppet to OpenVox Migration

## Why

Open Source Puppet packaging was discontinued upstream. OpenVox, maintained by Vox Pupuli, is the community fork and a drop in replacement at the code level. Roles, profiles, Hiera data, and r10k all carried over unchanged.

## Approach

Build new, migrate, then retire. The old primary and PuppetDB VMs kept running until every node was confirmed healthy on the new primary.

1. Stood up openvox-cli with OpenVox server, OpenVoxDB, and PuppetBoard on one VM
2. Pointed r10k at the same Gitea repo and production branch
3. Moved each agent to the new server, letting autosign issue new certs
4. Confirmed facts, catalogs, and reports flowing into OpenVoxDB for every node
5. Took backups of the eyaml keypair and a database dump off the cluster
6. Hibernated the old primary and PuppetDB, then deleted them once things were stable

## Issues found and fixed

| Issue | Fix |
|---|---|
| Server could not find code | Set `server-code-dir` in puppetserver.conf to the /etc/puppet/code path |
| Facts and catalogs not stored | Added a routes.yaml so facts and catalogs use the OpenVoxDB terminus |
| PuppetBoard metrics returned 403 | Opened metrics access in auth.conf |
| PuppetBoard broke when metrics were unavailable | Patched index.py to fall back gracefully on 403 |
| Daily chart counts were off near midnight | Fixed a UTC versus Pacific day boundary bug in dailychart.py |
| DNS config rendered wrong | Fixed a tilde prefix bug in base::dns |
| Agent config hardcoded the old server | Parameterized base::puppet_agent_config and base::puppetdb_conf for per node overrides |
| Wildcard autosign | Replaced `*.lab.local` with an explicit allowlist |

The PuppetBoard and server side fixes are tracked in `puppetboard-fixes.md`. Setup and fix scripts live in the `docs/` folder of the internal homelab-puppet repo.

## Result

One primary, one database, one dashboard. Every managed node reports in, and the old hosts are gone.
