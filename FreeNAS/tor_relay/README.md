[ [<< Back to Main Menu](https://github.com/seth586/guides) ]

## Tor Relay

While it is safe and realtively anonymous to run a relay and a hidden service from the same internet connection, it isn't perfect. Do not run a relay from the same internet connection as your bitcoin & lightning jail if you require *absolute* anonymity! See [this](https://research.kudelskisecurity.com/2013/09/04/dont-run-a-tor-router-and-a-hidden-service-from-the-same-connection/) for details. Your home router shold be beefy enough to handle 7,000+ connections and the tor preject recommends a minimum of 16 Mbit download and upload speed for relays.

Create a new jail, forward TCP port 9001 to this jail's IP address, and ssh in.

### Install and configure tor:
```
# pkg install tor ca_root_nss nano nyx
# rm /usr/local/etc/tor/torrc
# nano /usr/local/etc/tor/torrc
```
Edit the configuration files for tor (I recommend setting up a burner email you can check once in a while):
```
#change the nickname "myNiceRelay" to a name that you like
Nickname myNiceRelay
ORPort 9001
ControlPort 9051
CookieAuthentication 1
ExitRelay 0
SocksPort 0
BandwidthRate 16 Mbits
BandwidthBurst 64 Mbits
MaxAdvertisedBandwidth 16 Mbits
# Change the email address bellow and be aware that it will be published
ContactInfo tor-operator@your-emailaddress-domain
Log notice syslog
```
Set `bandwidthrate` below the lower value of your download and upload speed. So if your ISP provides 500 Mbit download and 
250 Mbit upload, do not use any value over 250 Mbit. Check your speed [here](https://beta.speedtest.net/).

Save (Ctrl+o, ENTER) and exit (Ctrl+x)

### Set up auto updates:
```
# nano /root/pkg_upgrade.sh
```
Enter the following script:
```
#!/usr/bin/env sh
PATH="/bin:/usr/bin:/sbin:/usr/sbin"
RAND=$(jot -r 1 300)
sleep ${RAND}
env AUTOCLEAN=YES ASSUME_ALWAYS_YES=YES HANDLE_RC_SCRIPTS=YES pkg upgrade
```
Save (Ctrl+o, ENTER) and exit (Ctrl+x)

### Make executable and schedule the job to run:
```
# chmod +x /root/pkg_upgrade.sh
# echo "0 0 * * * root /bin/sh /root/pkg_upgrade.sh >/dev/null" >> /etc/crontab
# service cron restart
```
### Enable random IP_IDs (see [this](https://mebsd.com/freebsd-security-hardening/protecting-freebsd-with-sysctl-101.html))
```
# echo "net.inet.ip.random_id=1" >> /etc/sysctl.conf
```

Reboot your jail and ssh back in. `ps aux` should show tor running!

### Nyx

Lets use a terminal UI to monitor the useage of our relay!

```
# nyx
```
To exit, press (Ctrl+C)

It will take about ~3 hours for your relay to propogate through the network. Search for your node here: https://metrics.torproject.org/rs.html

It takes about a good two weeks before you will see steady traffic, see this tor project blog post [here](https://blog.torproject.org/lifecycle-new-relay).

[ [<< Back to Main Menu](https://github.com/seth586/guides) ]

