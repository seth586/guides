[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - **[Jellyfin]** - [ [aria2](3_aria2.md) ] - [ [Medusa](4_medusa.md) ]

### Create User:Group, Dataset, and apply permissions:
Goto your freenas UI, and create new group `jellyfin` under `Accounts / Groups / Add`:
```
GID = 710
name = jellyfin
```
Click `SUBMIT`

Create new user `jellyfin` under `Accounts / Users / Add`:
```
Full Name = jellyfin
Username = jellyfin
User ID = 710
New Primary Group = N
Primary Group = jellyfin
Home Directory = /nonexistent
Disable Password = Yes
Shell = nologin
```
Click `SUBMIT`. It is important that your uid & id are `710`, so go back and make sure they are correct!

Create your media dataset, and apply `jellyfin:jellyfin` as `user:group` recursively for your dataset

SSH into your `jellyfin` jail:
```
iocage console jellyfin
pw group add jellyfin -g 710
pw user add jellyfin -c jellyfin -u 710 -d /nonexistent -s /usr/bin/nologin
pw group mod jellyfin -m jellyfin
cat /etc/passwd
cat /etc/group
```

https://github.com/Thefrank/jellyfin-server-freebsd/blob/main/Installation_TrueNAS_GUI.md

[jailip]:8096 !
