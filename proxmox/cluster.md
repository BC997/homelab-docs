# Proxmox Cluster: As Built Reference

## Overview

| Property | Value |
|---|---|
| Proxmox version | 8.4 |
| Nodes | 4 |
| Votes | 4 |
| Quorum | 3 |

The cluster ran five nodes until September 2026. The fifth node was removed cleanly, its VMs were moved, and the hardware now runs as a standalone Ubuntu and Windows workstation outside the cluster.

## Nodes

| Node | IP | Notes |
|---|---|---|
| Apricot | 10.0.0.101 | General compute |
| Butternut | 10.0.0.102 | Stays powered on at all times for quorum margin |
| Cupcake | 10.0.0.103 | General compute |
| Dessert | 10.0.0.104 | Media workload, NIC fix applied (see below) |

## VM placement

| VM | Name | Node | Role |
|---|---|---|---|
| 101 | openvox-cli | Apricot | OpenVox primary, OpenVoxDB, PuppetBoard |
| 109 | plex-cli | Dessert | Plex Media Server |
| 110 | cli-docker | Cupcake | Docker host |
| 112 | HAOS | Butternut | Home Assistant OS |

Templates and lab utility VMs are not listed.

## Networking

HA managed VMs attach to `vmbr0`, which exists on every node, so they can migrate freely. plex-cli intentionally stays on a bridge tied to Dessert's hardware and is not expected to move.

For live network changes on a node, `ifreload -a` is used instead of a full networking restart. It is lower risk for VMs that depend on the bridge.

## NIC tuning on Dessert

Dessert uses an Intel e1000e NIC (eno2). In August 2026 a transmit ring hang on that NIC dropped corosync and triggered a cluster wide reboot (see `lessons-learned.md`). The fix disables segmentation and receive offloads:

    ethtool -K eno2 tso off gso off gro off

The setting is persisted in `/etc/network/interfaces` so it survives reboots. Earlier tuning also raised the TX ring size and disabled flow control on the same NIC.

## Update timers

`pve-daily-update.timer` defaults to 1 AM with up to five hours of random delay. Every node runs it at 4 AM with no delay through a systemd drop in:

    /etc/systemd/system/pve-daily-update.timer.d/override.conf

    [Timer]
    OnCalendar=
    OnCalendar=*-*-* 04:00:00
    RandomizedDelaySec=0

The empty `OnCalendar=` line clears the default before setting the new one.

## Storage

Shared NFS storage on a Synology NAS backs HA managed VM disks so they can move between nodes. The media share is separate and covered in `networking/nfs.md`.

## Diagnostics

* `pvecm status` for quorum and membership
* `ha-manager status` for HA state
* `journalctl --no-pager | cat` on each node, then compare timestamps across nodes to tell a single node event from a fleet wide one
