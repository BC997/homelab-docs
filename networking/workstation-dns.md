# Workstation DNS for Lab Names

## Problem

The main workstation runs a commercial VPN client. With the VPN connected, every lab host was reachable by IP, but `*.lab.local` names did not resolve.

## Diagnosis

1. `resolvectl status` showed the VPN interface claiming every lookup with a catch all routing domain (`~.`), so lab names went to the VPN provider's DNS servers.
2. Adding a more specific rule (`~lab.local` pointed at Pi-hole) did not help. A raw DNS query to Pi-hole failed locally with "Operation not permitted," which means the packet never left the machine.
3. The firewall rules showed why: the VPN client installs explicit rules that drop DNS to private address ranges over both TCP and UDP. Its LAN access and allowlist settings do not override them. This is deliberate leak protection.

## Fix

Lab names live in a labeled block in the workstation's `/etc/hosts`, mirrored from Pi-hole's Local DNS records. The system checks the hosts file before any DNS server, so the VPN rules never come into play. VPN settings stay at their defaults, and every other lookup still goes through the VPN.

## Upkeep

When a Pi-hole record is added or changed, the same line goes into the hosts block. Automating that is on the list alongside managing Pi-hole records in code.
