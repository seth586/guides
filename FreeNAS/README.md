**[Intro]** - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ] - [ [Extras](extras.md) ]

## Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

![FreeNAS_Jail](images/BTCBSDsmall.png) 

### Intro

I have been running a FreeNAS server for a few years now, and have come to appreciate what it offers as a personal home server. It is infamous for media streaming & aggregation, and file hosting.

## Why FreeNAS? Raspberri Pis are much cheaper!

Answer: [Don't be this guy](https://github.com/lightningnetwork/lnd/issues/1214)

Raspberri pis are awesome, they are cheap, consume tiny amounts of electricity, and a ton of documentation exists for so many projects, including bitcoin and lightning. The problem with Raspberri Pis are the hard drives on these setups. Lightning especially needs to be online at all times, and Pis are 1 spinning platter away from catastrophic failure. In addition, lightning does not have a reliable backup and restore system, yet. Disk corruption could result in lost channels and lost funds. 

FreeNAS is special because of the hard drive redundancy features. FreeNAS utilizes the resilient and modern ZFS file system, which not only adds redundancy, but hashes data on your drives to detect and automatically fix errors. ZFS is better than hardware RAID, which is [obsolete](http://newsvideo.su/tech/video/102062)! If you follow the [FreeNAS documentation](https://www.ixsystems.com/documentation/freenas/), you will be set up to automatically run SMART tests on your hard drives, scrub data to verify & fix disk errors, and receive email alerts if a drive begins failing on you, allowing you to insert a new drive and resliver without any downtime. Add a battery backup, set up UPS monitoring & emails and you are running an enterprise grade environment at home!

FreeNAS is based on FreeBSD, a UNIX style operating system similar to Linux. FreeBSD utilizes a jail system to seperate operating environments, similar to how virturalization works. Except jails are much more efficient and less resource intensive than virturalizing. For example, my server has seperate jails for plex, medusa, transmission, SoftEther, bitcoin core&electrum-personal-server&lnd, nextcloud, etc. If I 'mess up', its easy to nuke the jail and start over, without ever damaging the host system or other jails.

### What is the build cost of a FreeNAS system?
You can buy older generation servers on ebay for dirt cheap! If you want something "price is not a problem" new for bitcoin, nextcloud, plex + transcoding, here is a buy list:
Latest Generation:

$230 Motherboard:  [SUPERMICRO MBD-X11SSM-F-O Micro ATX Server Motherboard LGA 1151 Intel C236](https://www.newegg.com/Product/Product.aspx?Item=N82E16813183013)

$208 ECC RAM(x2):       [Supermicro MEM-DR480L-SL01-EU24 8GB (1x8GB) DDR4 2400 (PC4 19200) ECC Unbuffered Memory RAM](https://www.newegg.com/Product/Product.aspx?Item=9SIA7S67Y98853)

$215 CPU:           [Intel Xeon E3-1220 V6](https://www.newegg.com/Product/Product.aspx?Item=N82E16819117790)
$440 Mass Storage(2+raidz2=4 or 4+raidz2=6):       [WD Red 4TB NAS Hard Disk Drive](https://www.newegg.com/Product/Product.aspx?item=N82E16822236599)

$80 Power Supply:   [Any Seasonic Brand with 8 or more SATA power cables.](https://www.newegg.com/Product/Product.aspx?Item=9SIADZJ5W07067)

$110 Case:          [Fractal Design Node 804](https://www.newegg.com/Product/Product.aspx?Item=N82E16811352047)

Total: $1283

#### Whoa, I don't want to spend that much!
Thanks to our 21st century craving for all things digital, last generation hardware is being sold for pennies on the dollar! Just search ebay for "server Xeon E3 V3" and you can find fully equipped systems for $200-300, such as the HP Proliant ML310e Gen8 V2 or Dell T20 series.

Now compare the performance between a V3 and a V6 Xeon:
https://www.cpubenchmark.net/compare/Intel-Xeon-E3-1220-v3-vs-Intel-Xeon-E3-1220-v6/2022vs3131

Yeah, you can save a lot of $ running last gen used server gear!

### Requirements
So, at this point we can assume that you built your home server. Hopefully you were smart enough to follow the [hardware recommendation guide](https://forums.freenas.org/index.php?resources/hardware-recommendations-guide.12/). My basic recommendation is this: Make sure you get a server class motherboard that has Internet Protocol Management Interface (IPMI) & have Error Code Correcting (ECC) ram. I highly recommend 6 hard drives in RAIDZ2 configuration, it is the best space and redundancy for the money. Any amount of drives in RAIDZ1 loses redundancy the moment you have a hard drive failure, and 4 drives in RAIDZ2 only has half the storage capacity of 6 drives in RAIDZ2. If the value proposition is getting pricey, start with smaller hard drives. You can’t add drives to a volume once its setup, however you can replace drives with larger drives, and once all 6 drives are the larger size, you get to increase the size of the volume.

### Assumptions
I am assuming you know your way around your router. My example router is a Linksys WRT1900ACv1 running OpenWRT. Your router configuration user interface may be different than explained here.

Lets also assume that you installed FreeNAS on your home server (Version 11.2), navigated the [FreeNAS forums](http://forums.freenas.org/index.php), read the [FreeNAS documentation](https://www.ixsystems.com/documentation/freenas/), and set up a ZFS volume. Make sure you set up your SMART test, scrub schedule and email alerts!

Within this guide, any time a command line is represented by a single `#` hash, that represents the command line as root user inside your bitcoin jail. Any commands outside this definition are represented by their full path, which may differ from what you see see based on how you named your server. Hopefully the guide is clear enough. If not, PLEASE reach out to me!

### Goal
By the end of this guide, we will have bitcoin core compiled, serving connections over IP and tor. We will install electrum-personal-server, so we can use a hardware wallet to cold store our bitcoin savings, verified with our own node. We will have lightning lab's lnd implementation to onbard the lightning network, and we will use Ride The Lightning web user interface to manage our lnd server, as well as install the joule browser extension and connect it to our lnd server. Electrum, Joule, and Ride The Lightning will be usable remotely over tor hidden services.

### Methodology
There is more than 1 way to skin a cat. These are the preferred methods followed in this guide that may differ from other guides:

1. Minimize software requirements. This guide does not use systemd, which is a monolithic layer that acts between the kernel and the user space. It has its place, but we don't need it. I don't have an [opinion](https://muchweb.me/systemd-nsa-attempt/) on the matter, but FreeBSD's own daemon has enough functionality to act as our process monitors.

2. Minimal configuration. This guide is a baseline to get setup. Whenever a configuration file is referenced, follow the supporting docs to explore further configuration options.

### Recommendations
Use a password manager to keep track of all the passwords required to run FreeNAS and your software. It's good cypherpunk habit to use unique strong passwords with 3rd parties, too. KeePassDX is an encrypted open source password manager that runs on android. It can generate strong passwords for you. 

### Personal Notes
This guide is written not only to benefit others, but myself as well. Sometimes I don't touch my server for months on end, and forget how I set things up or did things. This guide is my attempt to act on my belief in the [Cypherpunk Manifesto](https://www.activism.net/cypherpunk/manifesto.html). If cypherpunks can't write code, then cypherpunks deploy code.

This guide will be kept up to date. 

### Contact me
If you have any trouble with this guide, or want to share something to improve the guide, contact me! No question is too dumb! I'd rather help people deploy code than waste time browsing social media!

Twitter: [Seth586](https://twitter.com/seth586)

Gab: [Seth586](https://gab.com/seth586)

Email: seth586@protonmail.com

### Give me an unreasonably small tip!
https://tippin.me/@seth586

### Shout outs
Special thanks to the Stadicus raspberri pi guide for inspiring this freebsd guide, check it out here:
https://github.com/Stadicus/guides/blob/master/raspibolt/README.md

Next: [ [Jail Creation](freenas_1_jail_creation.md) ]
