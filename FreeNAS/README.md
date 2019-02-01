**[Intro]** - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Intro

I have been running a FreeNAS server for a few years now, and have come to appreciate what it offers as a personal home server. It runs plex, medusa, and transmission for media consumption & transcoding, aggregation, and downloading. It runs a VPN so I can have access to my local area network on the road. It runs my own cloud service on nextcloud, which in addition to hosting my files, also syncs my calendar and contacts between devices. I can run a rooted android phone with LineageOS and NO google. It's a cypherpunk device. 

FreeNAS is special because of the hard drive redundancy features. While I appreciate all the raspberri pi documentation out there for bitcoin and lightning, the problem is the hard drive on these setups. Lightning especially needs to be online at all times, and were running hardware that is 1 spinning platter away from catastrophic failure. FreeNAS utilizes the ZFS file system, which not only adds redundancy, but hashes data on your drives to detect and fix errors, such as bitrot. If you follow the FreeNAS documentation, you will be set up to automatically run SMART tests on your hard drives, scrub data to verify & fix data integrity, and receive email alerts if a drive begins failing on you, allowing you to insert a new drive and resliver without any downtime. Add a battery backup, and you are running an enterprise grade environment at home!

FreeNAS is based on FreeBSD, a UNIX style operating system similar to Linux. FreeBSD utilizes a jail system to seperate operating environments, similar to how virturalization works. Except jails are much more efficient and less resource intensive than virturalizing. For example, my server has seperate jails for plex, medusa, transmission, SoftEther, bitcoin core&electrum-personal-server&lnd, nextcloud, etc. If I 'mess up', its easy to nuke the jail and start over, without ever damaging the host system.

### Presumptions and Requirements
So, at this point we can assume that you built your home server. Hopefully you were smart enough to follow the [hardware recommendation guide](https://forums.freenas.org/index.php?resources/hardware-recommendations-guide.12/). My basic recommendation is this: Make sure you get a server class motherboard that has Internet Protocol Management Interface (IPMI) & supports Error Code Correcting (ECC) ram. I highly recommend 6 hard drives in RAIDZ2 configuration, it is the best space and redundancy for the money. Any amount of drives in RAIDZ1 loses redundancy the moment you have a hard drive failure, and 4 drives in RAIDZ2 only has half the storage capacity of 6 drives in RAIDZ2. If the value proposition is getting pricey, start with smaller hard drives. You can’t add drives to a volume once its setup, however you can replace drives with larger drives, and once all 6 drives are the larger size, you get to increase the size of the volume.

I am assuming you know your way around your router. My example router is a Linksys WRT1900ACv1 running OpenWRT. Your router configuration user interface may be different than explained here.

Lets also assume that you installed FreeNAS on your home server, navigated the FreeNAS forums, read the [FreeNAS documentation](https://www.ixsystems.com/documentation/freenas/), and set up a ZFS volume. Make sure you set up your SMART test and scrub schedule and email alerts!

### Goal
By the end of this guide, we will have bitcoin core compiled, serving connections over IP and tor. We will have ncurses2 terminal user interface to monitor our bitcoin node. We will install electrum-personal-server, so we can use a hardware wallet to cold store our bitcoin savings, verified with our own node. We will have lightning lab's lnd implementation to onbard the lightning network, and we will use Ride The Lightning web user interface to manage our lnd server, as well as install the joule browser extension and connect it to our lnd server.

### Recommendations
Use a password manager to keep track of all the passwords required to run FreeNAS and your software. It's good cypherpunk habit to use unique strong passwords with 3rd parties, too. KeePassDX is an encrypted open source password manager that runs on android. It can generate strong passwords for you. 
