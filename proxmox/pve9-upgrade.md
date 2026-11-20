# Proxmox VE 8 to 9 Upgrade (November 2026)

A rolling, in place upgrade of all four cluster nodes from Proxmox VE 8.4.0 to 9.2, done one node at a time with HA maintenance mode so the important VMs never went down. This doc is the project summary: why it was needed, how it was planned, what broke along the way, and what changed so the cluster never falls this far behind again.

## Why

All four nodes were still on Proxmox VE 8.4.0 with kernel 6.8.12-9 from March 2025, about 239 packages behind. Nothing was installing updates on them. Proxmox VE 8 reached end of life on August 31 2026, so the kernel, QEMU and the cluster stack were no longer getting fixes. The upgrade had to cross a Debian release (bookworm to trixie), which rules out a simple package update.

## Result

| | Before | After |
|---|---|---|
| Proxmox VE | 8.4.0 | 9.2.31 |
| Debian | 12 bookworm | 13 trixie |
| Kernel | 6.8.12-9 | 7.0.14-30 |
| Pending packages | about 239 | 0 |
| Node security updates | none | automatic, daily |
| VM backups | none scheduled | weekly, keep last 2 |
| Proxmox alerts | mail to root, unread | phone and Home Assistant |

Quorum held for the whole upgrade. The OpenVox primary, Home Assistant and the Docker host (which runs DNS) moved between nodes live with about a one second pause. Only the two 8 core VMs that can't fit on the smaller nodes were stopped during their host node's turn. Secure Boot still works on the three nodes that use it, and the UPS shutdown setup survived on every node.

## How it was planned

The upgrade was scoped as phases, each with its own checks, so a problem would show up before it could reach the next node.

| Phase | Work |
|---|---|
| 0, prep | Backups of every VM, fix oversized VM memory, enable the guest agent everywhere |
| 0.5, cleanup | Remove a half built Ceph cluster that was never used (3 monitors, 0 disks) |
| 1, catch up | Bring every node to the last 8.4 release and its kernel, then soak for two days |
| 2 prep | Clear the one hard failure from `pve8to9`, install CPU microcode, pin NIC names |
| 2, upgrade | Debian 13 and Proxmox VE 9, one node at a time |
| 3, cleanup | Patching, backups, alerts, and loose ends found along the way |

Every node went through the same guarded block before its upgrade: `pve8to9 --full` with zero failures, the new keyring in place, HA maintenance mode on, the repo switch, `apt update`, and a dry run that stopped if anything critical (the cluster stack, QEMU, the kernel, `sudo`) was on the removal list. The upgrade itself ran interactively in tmux so a dropped SSH session couldn't kill it, and a second guarded block after the reboot checked quorum, time sync, mail, the UPS client, and the VMs before the next node started.

### Node order and what moved

| Node in maintenance | What happened | Downtime |
|---|---|---|
| Apricot | OpenVox primary moved live to Butternut | about 1 s pause |
| Butternut | Home Assistant moved live to Cupcake | about 1 s pause |
| Dessert | Its 8 core VMs were stopped, nothing moved | those VMs only |
| Cupcake | 8 core VMs stopped, Docker host moved live to Dessert | those VMs only; DNS stayed up |

This same table is now the playbook for monthly updates.

## What went wrong, and the fixes

* **HA tried to move an 8 core VM onto a 4 thread node.** During Phase 1, Plex was sent to a node with only four CPU threads and failed to start in a retry loop. HA checks neither RAM nor CPU threads when it picks a target. Plex and the other 8 core VM now stop during their host's turn, and the HA rule for the Docker host only lists the two 8 thread nodes.
* **The Ceph purge left its config behind.** `pveceph purge` reported a foreign monitor address and kept the config files. They were removed by hand after checking nothing used them. The Ceph client packages stay for the storage tools that need them.
* **`apt autoremove` wanted to remove `sudo`.** After the Ceph purge, autoremove listed `sudo` as unused. It was not run. `sudo` was marked as manually installed instead, and the rule now is to never run autoremove on a node without reading the list.
* **`pve8to9` failed on the systemd-boot meta package.** The nodes boot through the signed shim and GRUB, so the package was confirmed unused and removed.
* **The NIC pinning tool refused the current interface names.** Proxmox 9's pinning tool wants to rename interfaces to its own scheme. Instead, small hand written systemd `.link` files pin each node's current name to its MAC address, so no network config had to change. The pin was proven through the new 7.0 kernel on all four nodes.
* **A wait loop hung.** A script waited on the guest agent of a VM that didn't have the agent turned on. The agent was enabled and every wait loop now has a time cap.
* **The Docker host came back with new SSH host keys.** A memory resize restart gave the VM a new cloud-init instance ID, so cloud-init ran its first boot steps again and regenerated the keys. The new fingerprint was verified through the guest agent before trusting it. cloud-init is now disabled inside every VM and its config drive is being removed.

## Phase 3: cleanup

* **Node patching.** Debian security updates now install automatically every day on all four nodes through `unattended-upgrades`, limited to the Debian security origin, with no automatic reboots and no automatic package removal. Proxmox packages and kernels are updated by hand once a month with the playbook above.
* **Weekly VM backups.** A vzdump job runs Sundays at 02:15 for every VM, in snapshot mode with zstd, to the shared NAS storage, keeping the last two.
* **Proxmox alerts reach the phone.** A Proxmox webhook notification target and a catch all matcher send every notification (backup results, package updates, fencing, replication) to a Home Assistant webhook. Each one becomes a phone alert and an Alert history item. Tested from all four nodes.
* **Monthly reminder.** Home Assistant reminds on the first Sunday of each month to run the node updates and adds an open item to check off when done.
* **HA groups became HA rules.** Proxmox 9 converted the old HA groups to node affinity rules automatically, with the same priorities. Failback stays on.
* **Old snapshots and orphaned disks removed.** June golden image snapshots with saved RAM were deleted. Disks from three VMs whose configs were lost when a node left the cluster (including the retired Puppet server) were found on the NAS and removed. About 75 GB was reclaimed.
* **Monitoring.** A scrape target for the removed fifth node, failing since September, was taken out of Prometheus. pve-exporter and its alerts were confirmed working on Proxmox 9.

## Related work done in the same pass

* **Container updates.** The Docker host was still running the original Watchtower, which was archived in December 2025. It only worked because its compose file pinned an older Docker API version, so the next Docker release that dropped that API would have stopped all container updates without warning. It was swapped for the maintained fork after a monitor only dry run, and it now sends a report to Home Assistant after every weekly run, with a separate alert if a run never reports in. See `docker/architecture.md`.
* **A Home Assistant automation that had been skipping silently.** In HA 2026.11, HACS became the built in Marketplace and its own update entity disappeared. The weekly HACS update automation still listed that entity, and one missing entity makes a state condition fail as a whole, so the automation had been skipping every Sunday with no error anyone could see. The entity was removed from the list. See `haos/architecture.md`.
* **Alerts became a to do list.** Alert history entries used to be saved already checked off. They now stay open until checked off by hand, and the daily cleanup only removes checked items.

## Still open

* Remove the old 6.8 kernels once 7.0 has run for a while
* Compact the Docker host's disk file, which grew from an old snapshot
* Keep `cloud-init.disabled` in place on every VM through OpenVox
* Give corosync its own second network link
