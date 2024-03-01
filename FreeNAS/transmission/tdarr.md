To run Tdarr, we need a bhyve VM running ubuntu. 

NFS security notes:
https://nfs.sourceforge.net/nfs-howto/ar01s06.html

Set up a NFS share of your media folder, and limit access to your ubuntu server ip address. SSH into your ubuntu server.
```
$ sudo apt install nfs-common
$ showmount --exports 192.168.84.85 # <-- your NFS server IP address
$ sudo mkdir /mnt/nfs/media
$ sudo ls -lh /mnt/nfs/media # <-- permission should be denied, this is good! Lets create a user with access
$ sudo ls -lh /mnt/nfs       # <-- note the UID and GID of the media folder, these need to match our new user and group, example 810 810
$ sudo groupadd -g 810 mediausers
$ sudo useradd mediauser -M -u 810 -g 810 # <-- '-M' prevents login
$ ls -lh /mnt/nfs            # <--  you should now see mediauser mediausers as user and group owners!
$ sudo nano /etc/pam.d/su    # <-- allow your default admin user to access mediauser owned filesystems
auth sufficient pam_rootok.so
auth  [success=ignore default=1] pam_succeed_if.so user = mediauser
auth  sufficient                 pam_succeed_if.so use_uid user = seth
```

create docker compose yaml:
```
mkdir $HOME/tdarr
mkdir $HOME/tdarr/server
mkdir $HOME/tdarr/configs
mkdir $HOME/tdarr/temp
nano $HOME/tdarr/docker-compose.yaml
```
Configuration notes: https://docs.docker.com/compose/compose-file/05-services/#environment

Whenever you see a `:`, left is host, right is docker container

When configured, start!
```
$ docker-compose up -d
```
Open your browser to your TrueNAS bhyve VM ubuntu's ip_address:8265
