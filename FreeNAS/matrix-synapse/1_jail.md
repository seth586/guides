[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ **Jail Creation** ] - [ [Postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Jail & Dataset Plan

Before we start installing stuff, lets make a few datasets to keep important files in case you need to nuke and rebuild the jail. This is also useful to backup critical components should you decide to take advantage of OpenZFS snapshotting and back these up to another machine.


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
### Create Jails

Create two jails:

database jail - synapsedb - 192.168.84.78

synapse jail - synapse - 192.168.84.79

### Mount datasets to jail
```
synapse: /mnt/volume1/apps/synapse/config -> /mnt/volume1/iocage/jails/synapse/root/usr/local/etc/matrix-synapse
synapse: /mnt/volume0/apps/synapse/media_store -> /mnt/volume1/iocage/jails/synapse/root/var/db/matrix-synapse/media_store
synapsedb: /mnt/volume1/apps/synapse/db -> /mnt/volume1/iocage/jails/synapsedb/root/var/db/postgres/data16
```
### Start Jails

### Files that need backup to sucessfully restore your homeserver
These locations may vary from the default installation & configuration locations to streamline your dataset backups 
#### 1. Config files: 
```
/usr/local/etc/matrix-synapse/homeserver.yaml
/usr/local/etc/matrix-synapse/domain.tld.log.config
/usr/local/etc/matrix-synapse/domain.tld.signing.key
```
How to verify locations: 
```
# cat /usr/local/etc/rc.d/synapse | grep synapse_conf
# cat /usr/local/etc/matrix-synapse/homeserver.yaml | grep domain.tld.log.config
# cat /usr/local/etc/matrix-synapse/homeserver.yaml | grep signing_key_path
```
#### 2. Database folder: 
```
/var/db/postgres/data13
```
How to verify database location:
```
# sudo -i -u postgres
# psql
# show data_directory;
# exit
# exit
```
#### 3. Media Repo folder: 
```
/var/db/matrix-synapse/media_store
````
Non critical, worst case scenario historical chats will loose uploaded media & files

How to verify media repo location: `cat /usr/local/etc/matrix-synapse/homeserver.yaml | grep media_store_path:`




