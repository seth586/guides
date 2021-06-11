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

Database: `/var/db/postgres/data13`
```
sudo -i -u postgres
psql
show data_directory;
exit
exit
```

Config: `/usr/local/etc/matrix-synapse/homeserver.yaml & /usr/local/etc/matrix-synapse/log.config`

`cat /usr/local/etc/rc.d/synapse | grep synapse_conf`

Signing Key: `/var/db/matrix-synapse/example.tld.signing.key`

`cat homeserver.yaml | grep signing_key_path`

Media Repo: `/var/db/matrix-synapse/media_store` -> non critical, worst case scenario historical chats will loose uploaded media & files

`cat homeserver.yaml | grep media_store_path:`

### Create Jail

### Mount datasets to jail

### Start Jail
