# NFS: As-Built Reference

## Overview
The Synology DS923+ hosts the primary media share via NFS.
Two VMs mount this share: Plex (read-only) and cli-docker (read-write for arr stack).
qby also mounts the share for download client output.

## NFS share
| Property | Value |
|---|---|
| Host | Synology DS923+ |
| IP | 10.0.0.35 |
| Share path | /volume1/PlexMediaServer |
| Mount point (cli-docker) | /mnt/nas/plexmediaserver |
| Mount point (qby) | /mnt/nas/plexmediaserver |
| Mount point (Plex) | /mnt/nas/plexmediaserver |

## fstab entries

### cli-docker (10.0.0.80)
Managed by Puppet via profile::nfs_media:

    10.0.0.35:/volume1/PlexMediaServer /mnt/nas/plexmediaserver nfs rw,defaults,_netdev,vers=3 0 0

### qby (10.0.0.147)
Manually managed — not in Puppet:

    10.0.0.35:/volume1/PlexMediaServer /mnt/nas/plexmediaserver nfs defaults,_netdev,nofail,x-systemd.automount 0 0

### Plex (10.0.0.155)
Manually managed — not in Puppet:

    10.0.0.35:/volume1/PlexMediaServer /mnt/nas/plexmediaserver nfs defaults,_netdev,nofail,x-systemd.automount 0 0

## Synology NFS permissions
Each client host has an explicit NFS rule in DSM:
Control Panel -> Shared Folder -> PlexMediaServer -> Edit -> NFS Permissions

| Client | Privilege | Squash | Security |
|---|---|---|---|
| 10.0.0.80 | Read/Write | Map all users to admin | sys |
| 10.0.0.147 | Read/Write | No mapping | sys |
| 10.0.0.155 | Read/Write | Map all users to admin | sys |
| 10.0.0.0/24 | Read/Write | No mapping | sys |

Security must be set to sys only — adding krb5i breaks writes from non-Kerberos hosts.

## Synology folder permissions
The PlexMediaServer shared folder requires Everyone Read/Write applied at the
top level with Apply to this folder, sub-folders and files checked.

DSM -> File Station -> PlexMediaServer -> right-click -> Properties -> Permission
-> Everyone -> Read/Write -> Apply to this folder, sub-folders and files -> Save

If this permission is missing, arr stack imports and download client output
will fail with permission denied errors even though the NFS mount shows rw.

## Verify mount health
On any host:
    df -h | grep nas
    mount | grep nas

Test write access:
    touch /mnt/nas/plexmediaserver/test.txt && rm /mnt/nas/plexmediaserver/test.txt

## Synology permission reset issue
DSM updates, reboots, and backup jobs can reset PlexMediaServer folder
permissions back to read-only. Two automated tasks are in place to correct this:

| Task | Trigger | Time |
|---|---|---|
| Fix PlexMediaServer Permissions | Boot-up | On boot |
| Fix PlexMediaServer Permissions - Daily | Scheduled | 6:00 AM daily |

Tasks are configured in DSM -> Control Panel -> Task Scheduler.

## Advanced Share Permissions gotcha
DSM Advanced Share Permissions (Windows ACL mode) conflicts with standard
Linux/NFS permissions. When enabled, DSM enforces Windows-style ACL inheritance
which overrides any chmod or GUI permission changes.

Fix: Disable Advanced Share Permissions on the shared folder.
DSM -> Control Panel -> Shared Folder -> PlexMediaServer -> Advanced ->
uncheck Enable advanced share permissions.

## Puppet NFS mount point gotcha
The /mnt/nas/plexmediaserver directory resource in profile::nfs_media must NOT
have a mode set. Once the NFS share is mounted, chmod on the mount point fails
with "Operation not permitted" since the filesystem is owned by the NFS server.

Correct configuration in nfs_media.pp:

    file { '/mnt/nas/plexmediaserver':
      ensure  => directory,
      owner   => 'root',
      group   => 'root',
      require => File['/mnt/nas'],
    }

The mode attribute must be omitted. If present, Puppet will error on every run
trying to set permissions on the NFS mount point.
