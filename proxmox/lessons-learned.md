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
