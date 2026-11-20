# Proxmox Cluster: As Built Reference

## Overview

| Property | Value |
|---|---|
| Proxmox version | 9.2 (Debian 13 trixie) since November 2026 |
| Kernel | 7.0 |
| Nodes | 4 |
| Votes | 4 |
| Quorum | 3 |

The upgrade from Proxmox VE 8 is written up in `pve9-upgrade.md`.

The cluster ran five nodes until September 2026. The fifth node was removed cleanly, its VMs were moved, and the hardware now runs as a standalone Ubuntu and Windows workstation outside the cluster.

## Nodes

| Node | IP | CPU threads | Notes |
|---|---|---|---|
| Apricot | 10.0.0.101 | 4 | General compute |
| Butternut | 10.0.0.102 | 4 | Stays powered on at all times for quorum margin |
| Cupcake | 10.0.0.103 | 8 | General compute |
| Dessert | 10.0.0.104 | 8 | Media workload, NIC fix applied (see below) |

VMs with 8 cores can only run on Cupcake or Dessert. HA checks neither CPU threads nor RAM when it picks a node, so the HA rules for those VMs only list the 8 thread nodes.

All four nodes boot UEFI through the signed shim and GRUB. Secure Boot is on for Apricot, Butternut and Cupcake.

## VM placement

| VM | Name | Node | Role |
|---|---|---|---|
| 101 | openvox-cli | Apricot | OpenVox primary, OpenVoxDB, PuppetBoard |
| 109 | plex-cli | Dessert | Plex Media Server |
| 110 | cli-docker | Cupcake | Docker host |
| 112 | HAOS | Butternut | Home Assistant OS |

Other lab VMs are not listed. Every VM has fixed memory (balloon equal to its size) and the guest agent enabled. cloud-init is disabled inside every VM, so a cold restart can't regenerate SSH host keys. No VM templates exist right now. Rebuilding a cloud init template is on the roadmap.

## HA rules

Proxmox 9 replaced HA groups with node affinity rules in `/etc/pve/ha/rules.cfg`. The old groups were converted automatically with the same node priorities, non strict, and failback stays on.

| VM | Preferred nodes, in order |
|---|---|
| OpenVox primary | Apricot, Butternut, Cupcake, Dessert |
| HAOS | Butternut, Cupcake, Apricot, Dessert |
| cli-docker | Cupcake, Dessert |
| plex-cli | Dessert, Cupcake |

The new status line `fencing armed (CRM watchdog active)` is normal on Proxmox 9.

## Networking

Every node has exactly one bridge, `vmbr0`, and every VM attaches to it. Any VM can run on any node.

For live network changes on a node, `ifreload -a` is used instead of a full networking restart. It is lower risk for VMs that depend on the bridge.

Interface names are pinned by hand with a small systemd `.link` file per node under `/usr/local/lib/systemd/network/`, matching the current name to the NIC's MAC address. Proxmox 9's own pinning tool would have renamed every interface, and the hand written pin kept the existing network config working unchanged through the kernel upgrade.

## NIC tuning on Dessert

Dessert uses an Intel e1000e NIC (eno2). In August 2026 a transmit ring hang on that NIC dropped corosync and triggered a cluster wide reboot (see `lessons-learned.md`). The fix disables segmentation and receive offloads:

    ethtool -K eno2 tso off gso off gro off

The setting is persisted in `/etc/network/interfaces` so it survives reboots. Earlier tuning also raised the TX ring size and disabled flow control on the same NIC.

## Patching, backups and alerts

* **Security updates, automatic.** `unattended-upgrades` installs Debian security updates daily on every node. A local config file clears the default origin list and allows only the Debian security origin, with automatic reboots and automatic package removal turned off. Proxmox packages and kernels can never match it.
* **Proxmox updates, monthly by hand.** One node at a time with HA maintenance mode, following the node order in `pve9-upgrade.md`. Home Assistant sends a reminder on the first Sunday of each month.
* **VM backups.** A vzdump job runs Sundays at 02:15 for every VM, in snapshot mode with zstd, to the shared NAS storage, keeping the last two.
* **Notifications.** A webhook notification target and a catch all matcher send every Proxmox notification to Home Assistant, which turns it into a phone alert. Test with `pvesh create /cluster/notifications/targets/<target>/test`.
* **Never run `apt autoremove` on a node without reading the list.** After the Ceph cleanup it wanted to remove `sudo`.

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
* `ha-manager crm-command node-maintenance enable <node>` to move a node's VMs off before work, and `disable` to bring them back
* `journalctl --no-pager | cat` on each node, then compare timestamps across nodes to tell a single node event from a fleet wide one

## Watch items

All four nodes use the same Intel e1000e NIC model. An October 2026 audit found zero error counters and zero transmit hangs on every node. Corosync still logs short drops between single pairs of nodes, spread across all hours of the day. Hardware and scheduled jobs were ruled out, which points to traffic spikes on the shared LAN. The long term fix is a dedicated second corosync link.
