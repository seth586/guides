[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
## Run LND on clearnet
To run LND on clearnet, we need to advertise our IP address. But what if our ISP changes our IP address? This guide explains how to set up your router and LND to use `nat=true` so LND will detect our public IP and broadcast changes to the network.

### Universal Plug & Play / Network Address Translation - Port Mapping Protocol

Universal Plug and Play is a standard to allow clients to configure the router's port mapping. It is a tradeoff of security for convenience. It is much more secure to manually set up port forwards in your router. For example, video game consoles may need to open and close ports on the fly, depending on the multiplayer game.

Most commercial grade routers with manufacturer firmware use a very [insecure implementation](https://www.howtogeek.com/122487/htg-explains-is-upnp-a-security-risk/), letting anyone that requests a port forward to do so! Imagine torjan software on your desktop, or your kids computer, reconfiguring your router! 

There are a few scenarios where a safe implementation of upnp can be reasonably secure, if we can control what has access to it.  Lightning Lab's `lnd` queries upnp or NAT-PMP for a current ip address assigned by your ISP. If a change is detected, your peers are notified and you can maintain connectivity with your inbound channels.

### So how can we securely use upnp or NAT-PMP?

If we can control what IP addresses can use upnp/NAT-PMP, we can minimize the vulnerability surface. 

So log in to your router, and see if your router's upnp or NAT-PMP implementation has an `access control list` to limit what client IP addresses can request a port mapping and what port ranges can be opened. If it doesn't, then disable upnp/NAT-PMP! 

I highly, highly, recommend you buy a router than can run OpenWRT, an open source router firmware. Its software is kept up to date, and can be installed on a large number of [consumer devices](https://openwrt.org/supported_devices). Many manufacturers do not offer continued support for their routers firmware, allowing security vulnerabilities to live on, unpatched.

### Goal

This guide is to help you install `miniupnpd` daemon with the `luci-app-upnp` user interface, configured to serve `lnd`'s requests for the external IP address on an OpenWRT compatible router. Fortunately `lnd` only queries the `miniupnpd` daemon for the external IP address, so we will set permissions to deny any upnp port forward request, and manually set up port forwards.

### Install miniupnpd

Log in to your OpenWRT router's web interface, typically at `192.168.X.1`, where `X` is unique to your local area network subnet.

Click `System`, then `Software`. Click `Update Lists`, then type in `luci-app-upnp`. Click `install` next to `luci-app-upnp`. The package and dependencies will install. 

You may have to manually enable autostart. Click on `System`, then `Startup`. Make sure `miniupnpd` is enabled. 

Click `services` and `UPnP`. 

Check the `Start UPNP and NAT-PMP service` box, `Enable NAT-PMP functionality`, `enable secure mode`. 

Delete the `default allow` rule under the `MiniUPnP ACLs`. 

Make sure there is a `default deny` rule: External ports `0-65535`, Internal Addresses `0.0.0.0/0`, Internal Ports `0-65535` Action `deny`.

Rules are read top to bottom, so if you have a game console or another service that requires upnp or nat-pmp, put the rule above the deny rule.

Click `save and apply`. 

Now would also be a good time to port forward 9735 for lnd:
```
Name : lnd
Protocol: TCP+UDP
External Zone: WAN
External Port: 9735
Internal Zone: LAN
Internal IP address: (inset your jail IP here)
Internal Port: 9735
```

Reboot your router.

### Configure `lnd` for `NAT-PMP`
SSH in to your FreeNAS box, and switch consoles to your bitcoin jail.
```
root@freenas:~# iocage console bitcoin
```

Edit your `lnd.conf` file:
```
# nano /home/bitcoin/.lnd/lnd.conf
```
Remove all tor entries except for outbound connections with `tor.active=1` and `tor.socks=localhost:9050`. Add `nat=true` under `[Application Options]`
Save (CTRL+O,ENTER) and exit (CTRL+X)

Stop the lnd service, and manually start it to read the verbose messages:
```
# service lnd stop
# lnd --configfile=/usr/local/etc/lnd.conf
```
 Unlock your wallet (If you installed RTL, you can do it there, otherwise start a new SSH session, switch to user bitcoin `su bitcoin` and run `lncli unlock`

Look for the following lines, you may have to grab the scroll bar on the right side of your SSH window, the messages move fast:
```
2019-02-12 18:11:02.150 [ERR] SRVR: Unable to discover a UPnP enabled device on the local network: context canceled
2019-02-12 18:11:02.150 [INF] SRVR: Scanning local network for a NAT-PMP enabled device
2019-02-12 18:11:02.183 [INF] SRVR: Automatically set up port forwarding using NAT-PMP to advertise external IP

```
Sucess! 

If it fails, `lnd` will exit. Double check on your router that `miniupnpd` is running under `status` / `processes`.

Press (CTRL+C) to stop the lnd process. Exit the `bitcoin` user shell, and start the `lnd` service.
```
bitcoin@bitcoin:~% exit
# service lnd start
```

[ [Back to LND setup guide](https://github.com/seth586/guides/blob/master/FreeNAS/freenas_5_lnd.md) ]

[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
