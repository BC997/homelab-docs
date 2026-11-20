# Lessons Learned

Each of these came from a real incident or a real mistake in this lab.

## Quorum margin matters more than node count

With five nodes and a quorum of three, the cluster often ran with exactly three nodes powered on. That is zero margin. One flaky NIC was enough to take the whole cluster down. One node now stays on permanently so the cluster always has a vote to spare.

## HA fencing is fast and it hits everything at once

The failure chain from the August 2026 incident:

1. A transmit ring hang on Dessert's e1000e NIC stalled its network traffic
2. Corosync lost contact and the cluster lost quorum
3. On each HA active node, pve-ha-lrm released its watchdog lock
4. About 60 seconds later the softdog timer expired and hard rebooted the node
5. Every HA active node did this at the same time

Corosync on Dessert had been flapping for weeks beforehand. The warning signs were in the logs. Corosync link flaps are now treated as an early warning, not noise.

## VM portability depends on the bridge

A VM attached to a bridge that only exists on one node cannot migrate, and it breaks if that node leaves the cluster. Removing a node exposed this, so every HA managed VM was moved to `vmbr0`, which exists on all nodes.

## Some steps are still manual, and manual steps drift

Pi-hole Local DNS records are not managed by Puppet. A new VM without a record fails name resolution in confusing ways. This is part of the onboarding checklist until it is automated.

## Puppet specifics

* **Not managing a resource is not the same as removing it.** If a flag turns a feature off, the profile needs an explicit `ensure => absent` branch, or the old resource stays in place.
* **Order exec guards carefully.** An `unless` guard that runs before the resource it checks can fire a corrective exec on every run. Fixing the dependency order made the runs clean.
* **Set the timezone in code.** A VM left on UTC ran a "nightly" cron in the early evening. Timezone is now enforced in the base profile.
* **A clean run is not a verified change.** Always check the service on the node itself.

## Working rules that came out of all this

* Back up before editing, test before touching live config, and keep changes reversible
* Validate by hand before trusting automation
* Delay cleanup, like revoking old credentials, until several automated cycles succeed
* You cannot shrink a mounted root partition from the running OS. Boot a live USB.

## Reading corosync link drops

When a node really goes down, every peer logs it at the same moment. When only one peer logs a drop and the others do not, the cause is a short blip on the network between that pair, not a dead node. Counting drops by day and by peer, then by hour, separates maintenance windows, scheduled jobs, and real network trouble before anything gets changed.

## Find every user of a credential before revoking it

The old Gitea token was assumed to belong to one service. An audit showed two other things on the primary used the same token: r10k, and the working clone through its git remote address. Each one was moved to its own SSH key and tested first. Revoking it then broke nothing. A one way change like a revocation comes last.

## A VPN client can block LAN DNS on purpose

A workstation could reach every lab host but could not resolve lab names. The VPN client's firewall had explicit rules dropping DNS to private address ranges, a leak protection that its LAN settings do not override. Lab names were moved into the workstation's hosts file instead of weakening the VPN. See `networking/workstation-dns.md`.

## HA does not check whether a VM fits

During the Proxmox 9 upgrade, HA sent an 8 core VM to a node with four CPU threads, and the VM sat in a start and fail loop. HA picks a node from its rules and priorities only. It checks neither CPU threads nor RAM. Rules for large VMs now list only the nodes that can run them, and the per node playbook stops those VMs instead of moving them when there is nowhere they fit.

## A pinned version can hide end of life software

The Docker host's update tool had been archived for almost a year and still looked healthy every week. It only worked because its compose file pinned an older Docker API version. The day Docker dropped that API, every container update would have stopped with no alert. A pin that keeps old software running is a deadline, so it gets written down next to the thing it is propping up.

## A missing entity can switch off an automation without an error

In Home Assistant, a state condition that lists several entities with `match: any` still fails as a whole if one of them no longer exists. When HACS became the built in Marketplace, its update entity disappeared and the weekly update automation quietly stopped installing anything. After a platform change, entity lists in conditions get rechecked, and HA's validation warnings on save are worth reading.

## Removed nodes leave disks behind

When a node leaves the cluster with VMs still assigned to it, their configs go with it, but their disks on shared storage stay. Nothing ever cleans them up because Proxmox no longer knows they exist. A storage audit against the list of current VMs found three of these, including the retired Puppet server.

## cloud-init can run again on an old VM

A restart after a memory change gave a VM a new cloud-init instance ID, so cloud-init ran its first boot steps again and regenerated the SSH host keys. The SSH warning that followed looked like a security problem. The new key was confirmed through the guest agent before it was trusted, and cloud-init is now disabled inside every VM after first boot.

## Read the removal list

Before every node upgrade, the dry run was checked for critical packages on the removal list. The same habit caught `apt autoremove` trying to remove `sudo` after the Ceph cleanup. Anything that removes packages on a hypervisor gets read line by line first.
