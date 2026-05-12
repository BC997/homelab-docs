# Proxmox Cluster: As-Built Reference

## Cluster overview

| Property | Value |
|---|---|
| Cluster name | homelab |
| Proxmox version | 8.4.0 |
| Nodes | 5 |
| Quorum votes required | 3 |

## Nodes

| Node | IP | Role |
|---|---|---|
| Apricot | 10.0.0.101 | General compute |
| Butternut | 10.0.0.102 | General compute |
| Cupcake | 10.0.0.103 | General compute |
| Dessert | 10.0.0.104 | General compute |
| Echo | 10.0.0.105 | High memory / media |

## VM inventory

| VM ID | Name | Node | State | CPU | RAM | Disk | IP |
|---|---|---|---|---|---|---|---|
| 100 | Ubuntu-ServerCLI-Template | Echo | stopped (template) | - | - | - | - |
| 101 | Plex | Echo | running | 16c | 64GB | - | 10.0.0.4 |
| 102 | Ubuntu-Desktop-Template | Echo | stopped (template) | - | - | - | - |
| 103 | Kali | Echo | stopped | 4c | 6GB | 80G | 10.0.0.7 |
| 104 | Super-Admin | Echo | stopped | 16c | 60GB | - | - |
| 105 | Windows-VM | Dessert | stopped (template) | - | - | - | - |
| 106 | puppet.lab.local | Butternut | running | 4c | 8GB | 64G | 10.0.0.2 |
| 107 | RBT | Echo | running | 8c | 32GB | - | 10.0.0.5 |
| 108 | puppetdb.lab.local | Apricot | running | 4c | 8GB | 45G | 10.0.0.6 |
| 110 | cli-docker.lab.local | Dessert | running | 8c | 8GB | 64G | 10.0.0.3 |
| 112 | HAOS | Cupcake | running | 8c | 16GB | 32G | 10.0.0.8 |

## NIC tuning

### Dessert (e1000e, eno2)

TX ring buffer raised and flow control disabled to prevent TX queue hangs.
A prior cluster-wide outage was traced to a TX queue hang on Dessert's e1000e NIC
causing Corosync quorum loss.

    sudo ethtool -G eno2 tx 4096
    sudo ethtool -A eno2 autoneg off rx off tx off

These settings are applied at boot via a systemd service or udev rule.

### Echo (r8169, enp42s0)

Flow control disabled. TX ring is capped at 256 by the r8169 driver — this cannot
be raised without a hardware change. An Intel I350 NIC upgrade is recommended
for Echo long-term.

    sudo ethtool -A enp42s0 autoneg off rx off tx off

## New VM onboarding workflow

For Desktop GUI Ubuntu VMs cloned from VM 102 (Ubuntu-Desktop-Template):

1. Set a DHCP reservation in the router for the new VM's MAC address
2. Clone VM 102 in Proxmox on the target node
3. Boot the clone and set the hostname (auto-populates /etc/hosts)
4. Run puppet agent -t (autosign handles cert issuance automatically)
5. Verify the node appears in PuppetBoard

## NFS

Media share hosted on a Synology NAS.
Mounted on Plex VM at /mnt/nfs/PlexMediaServer via fstab:

    <NAS_IP>:/volume1/PlexMediaServer /mnt/nfs/PlexMediaServer nfs defaults,_netdev 0 0

Do not modify, move, or delete any data on this NFS mount.
