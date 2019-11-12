[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

## Tor Relay

Do not run a relay from the same internet connection as your bitcoin & lightning jail if you require the anonymity! See [this](https://research.kudelskisecurity.com/2013/09/04/dont-run-a-tor-router-and-a-hidden-service-from-the-same-connection/) for details.

Create a new jail, forward port 9001 to this jail's IP address, and ssh in.

Install and configure tor:
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
ExitRelay 0
SocksPort 0
BandwidthRate 2500 KBits
MaxAdvertisedBandwidth 2500 KBits
# Change the email address bellow and be aware that it will be published
ContactInfo tor-operator@your-emailaddress-domain
Log notice syslog
```
Save (Ctrl+o, ENTER) and exit (Ctrl+x)

Set up auto updates:
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
Schedule the job to run:
```
# echo "0 0 * * * root /bin/sh /root/pkg_upgrade.sh >/dev/null" >> /etc/crontab
# service cron restart
```
Enable random IP_IDs (see [this](https://mebsd.com/freebsd-security-hardening/protecting-freebsd-with-sysctl-101.html))
```
# nano /etc/sysctl.conf
```
Add the following line:
```
net.inet.ip.random_id=1
```



[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
