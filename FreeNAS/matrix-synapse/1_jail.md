https://github.com/seth586/guides/blob/master/FreeNAS/matrix-synapse/1_jail.md

Before we start installing stuff, lets make a few data volumes to keep important files in case you need to nuke and rebuild the jail. This is also useful to backup critical components should you decide to take advantage of OpenZFS snapshotting and back these up to another machine.



Database: /var/db/postgres/data13
```
sudo -i -u postgres
psql
show data_directory;
exit
exit
```

Config: /usr/local/etc/matrix-synapse/homeserver.yaml & /usr/local/etc/matrix-synapse/log.config
`cat /usr/local/etc/rc.d/synapse | grep synapse_conf`

Signing Key: /var/db/matrix-synapse/example.tld.signing.key
`cat homeserver.yaml | grep signing_key_path`

Media Repo: /var/db/matrix-synapse/media_store
`cat homeserver.yaml | grep media_store_path:`
