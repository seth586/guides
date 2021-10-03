[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - **[Jellyfin]** - [ [aria2](3_aria2.md) ] - [ [Medusa](4_medusa.md) ]

Goto your freenas UI, and create a new user under `Accounts / Users / Add`:
```
Full Name = jellyfin
Username = jellyfin
User ID = 710
New Primary Group = Y
Home Directory = /nonexistent
Disable Password = Yes
Shell = nologin
```
Click `SUBMIT`

SSH into your `jellyfin` jail:
```
iocage console jellyfin
pw user add jellyfin -c jellyfin -u 710 -d /nonexistent -s /usr/bin/nologin
```
