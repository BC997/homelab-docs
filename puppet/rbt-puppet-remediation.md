# RBT Puppet Remediation

## Overview

RBT (VM 107, rbt.lab.local, 10.0.0.5) was audited in May 2026 and found to
have no functional Puppet enforcement despite having puppet-agent installed. All
NordVPN settings and the NFS mount were manually configured. This document covers
the remediation applied to bring RBT under full Puppet management.

## Issues Found

- Puppet binary not in PATH — agent installed but never run
- `role::media_server` missing `profile::nfs_media`
- `profile::nordvpn` only installed snap and ensured daemon — no settings enforced
- NFS mount at `/mnt/nas/plexmediaserver` was manual via fstab only

## Changes Made

### role::media_server
Added `include profile::nfs_media` to ensure the Synology NFS mount is managed
by Puppet on all media server nodes.

### profile::nordvpn
Added idempotent `exec` resources for all NordVPN settings. Each uses a `grep -F`
against `nordvpn settings` output as the `unless` guard to prevent re-application
on clean runs. The `lan-discovery` exec uses `returns => [0, 1]` to handle the
non-zero exit code the CLI returns when the setting is already enabled.

Settings enforced:
- Technology: NORDLYNX
- Firewall: enabled
- Kill Switch: enabled
- Auto-connect: enabled
- DNS: disabled
- Meshnet: disabled
- Threat Protection Lite: disabled
- LAN Discovery: enabled

Binary path: `/snap/bin/nordvpn` (NordVPN installed via snap on RBT)

## Validation

Drift detection confirmed — manually disabling Kill Switch and running
`puppet agent -t` corrected it back to enabled within a single run.

