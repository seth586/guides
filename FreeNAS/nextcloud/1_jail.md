[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ **Jail Creation** ] - [ [nginx](4_apache.md) ] - [ [PHP](3_php.md) ] - [ [mariadb](2_mariadb.md) ] - [ [nextcloud](5_nextcloud.md) ] - [ [reverseproxy ](6_reverseproxy.md)] - [ [collabora](7_collabora.md) ]

## Guide to Nextcloud server on TrueNAS

### Jail & Dataset Plan

Before we start installing stuff, lets make a few datasets to keep important files in case you need to nuke and rebuild the jail. This is also useful to backup critical components should you decide to take advantage of OpenZFS snapshotting and back these up to another machine.

### Create users
Users in the dataset outside the jail need to match user permissions inside the jail, so they must have matching username user ID (UID).

In TrueNAS' web-ui, click `accounts -> users -> add`, and create the following:
```
Username: mysql
Full Name: MySQL User
User ID: 88
New Primary Group: Checked
Enable Password login: No
```
When finished, click `Submit`

### Create datasets
I have two [pools](https://www.truenas.com/docs/core/storage/pools/poolcreate/), `volume0`, a RAIDZ2 pool of several HDDs and `volume1`, a mirrored pool of two high performance SSDs (Samsung PM1725b HHHL NVMe x8). [Datasets](https://www.truenas.com/docs/core/storage/pools/datasets/) will be created to store important data that needs to be backed up. 

I highly recommend the following dataset folder structure, as it will make sense as you deploy more software, example on my system:
```
─── volume0
    ├── apps
    │   ├── nextcloud
    │   │   ├── files  
    │   ├── synapse
    │   │   ├── mediastore 

─── volume1
    ├── apps
    │   ├── nextcloud
    │   │   ├── config
    │   │   ├── themes
    │   │   ├── db
    |   ├── synapse
    │   │   ├── config
    │   │   ├── signingkey
    │   │   ├── db    
... and so on
```  

Login to TrueNAS' web-ui, click `storage -> pools`, and create the following datasets based on your own pool configuration:
```
volume0/apps/nextcloud/files
Enable atime: Off

volume1/apps/nextcloud/db
Enable atime: Off

volume1/apps/nextcloud/config
Enable atime: On

volume1/apps/nextcloud/config/themes
Enable atime: On
```

### Assign dataset permissions
Click the `three-dot icon` next to each dataset you created, click `permissions` and assign the following permissions, clicking `apply user checkmark` and `apply group checkmark`, type in the user/group, then click `save`:
```
db
User: mysql
Group: mysql

config
User: www
Group: www

themes
User: www
Group: www
```

### Create Jail
Log into TrueNAS. Click `Jails > Add`. Create a jail with `VNET`, `allow_raw_sockets`, set `name` to `nextcloud`. `release`, and `IPv4 address`. Click `autostart` and click `SAVE`.

### Mount datasets to jail
ssh into TrueNAS. 
```
$ iocage stop nextcloud
$ iocage exec nextcloud mkdir -p /mnt/data
$ iocage exec nextcloud mkdir -p /var/db/mysql
$ iocage exec nextcloud mkdir -p /usr/local/www/nextcloud/config
$ iocage exec nextcloud mkdir -p /usr/local/www/nextcloud/themes
$ iocage fstab -a nextcloud /mnt/volume0/apps/nextcloud/files /mnt/data nullfs rw 0 0
$ iocage fstab -a nextcloud /mnt/volume1/apps/nextcloud/db /var/db/mysql nullfs rw 0 0
$ iocage fstab -a nextcloud /mnt/volume1/apps/nextcloud/config /usr/local/www/nextcloud/config nullfs rw 0 0
$ iocage fstab -a nextcloud /mnt/volume1/apps/nextcloud/themes /usr/local/www/nextcloud/themes nullfs rw 0 0
$ zfs set primarycache=metadata volume1/apps/nextcloud/db
$ iocage start nextcloud
$ iocage console nextcloud
```

