[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [**Jail Creation**] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ] 

##  TrueNASnode - full bitcoin stack deployment guide ![BSDBTC100.png](images/BSDBTC60.png)

Join the chatroom on the matrix chat protocol: [#truenasnode:nym.im](https://matrix.to/#/#truenasnode:nym.im)

### Jail Creation

Think of jails as more efficient virtual machines (VMs). You could just install a bunch of VMs on TrueNAS, run linux on them, and pick your choice on the many varieties of linux guides available online. But running a VM requires a lot more resources than jails, allocating memory just for that VM, etc. Plus, if we mess up, we can delete the jail and start over. Anything we do in the jail should not mess up anything on the host machine. After all, we built a computer with server grade hardware for the uptime!

TrueNAS uses iocage to manage jails. Previous versions used warden, which is now considered deprecated. To create a jail, log in to your TrueNAS user interface, and select Jails on the left hand menu. Click the ‘ADD’ button on the top right, and select 'advanced jail creation'.

![TrueNAS_Jail](images/jail_create.png)  

Give the jail a name, such as `bitcoin`. Select release `11.2-RELEASE`, select `DHCP Autoconfigure IPv4` and select `Auto-start` as shown. Click `SAVE`.

It would be a good idea to log into your router and give your bitcoin jail a static IP address. Also forward port 8333 from your WAN to your jail's LAN IP address. For example, my internal IP address assigned to my bitcoin jail is 192.168.84.123

![TrueNAS_Jail_Port_Forward](images/jail_port_forward.png)

*Your router's firmware may look different. This is how it looks on OpenWRT.*
```
Name : bitcoin
Protocol: TCP
External Zone: WAN
External Port: 8333
Internal Zone: LAN
Internal IP address: (inset your jail IP here)
Internal Port: 8333
```


Next: [ [Install Bitcoin](freenas_2_bitcoin.md) ]
