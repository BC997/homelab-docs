# NFS: As Built Reference

## Overview

A Synology DS923+ hosts the media share over NFS. Plex serves from it, and the cli-docker arr stack writes to it.

## Share

| Property | Value |
|---|---|
| Host | Synology DS923+ |
| IP | 10.0.0.20 |
| Share path | /volume1/PlexMediaServer |
| Mount point | /mnt/nas/plexmediaserver |

## Mounts

Both mounts are managed by Puppet.

| Host | Access | Profile |
|---|---|---|
| cli-docker | Read and write | profile::nfs_media |
| plex-cli | Read and write, so media can be removed from inside Plex | profile::plex |

cli-docker fstab entry rendered by Puppet:

    10.0.0.20:/volume1/PlexMediaServer /mnt/nas/plexmediaserver nfs rw,defaults,_netdev,vers=3 0 0

Data on this share is never moved, deleted, or modified by hand or by automation outside the apps that own it.

## Synology NFS permissions

DSM > Control Panel > Shared Folder > PlexMediaServer > Edit > NFS Permissions

| Client | Privilege | Squash | Security |
|---|---|---|---|
| cli-docker | Read/Write | Map all users to admin | sys |
| plex-cli | Read/Write | Map all users to admin | sys |
| 10.0.0.0/24 | Read/Write | No mapping | sys |

Security must be `sys` only. Adding krb5i breaks writes from hosts without Kerberos.

The service account uses UID 1026 and GID 100 to line up with the Synology side.

## Folder permissions

The shared folder needs Everyone Read/Write at the top level, applied to all sub folders and files. Without it, arr stack imports fail with permission denied even though the mount shows rw.

DSM updates, reboots, and backup jobs can reset these permissions. Two DSM scheduled tasks put them back, one on boot and one daily.

## Advanced Share Permissions gotcha

DSM Advanced Share Permissions (Windows ACL mode) overrides normal Linux and NFS permissions. It is turned off on this share.

## Puppet mount point gotcha

The mount point directory in `profile::nfs_media` must not set a mode. Once the share is mounted, chmod on the mount point fails because the NFS server owns it, and Puppet errors on every run.

    file { '/mnt/nas/plexmediaserver':
      ensure  => directory,
      owner   => 'root',
      group   => 'root',
      require => File['/mnt/nas'],
    }

## Health checks

    df -h | grep nas

    mount | grep nas
