[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](1_jail.md) ] - [ [Postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - **[ token registration ]** - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ]- [ [jitsi](8_jitsi.md) ] - [ [bridges](9_bridges.md) ]

## Guide to matrix-synapse server on TrueNAS ![BSDBTC60.png](images/matrix60.png)

### Token based registration bot
https://github.com/moan0s/matrix-registration-bot

#### Enable token registrations on your homeserver
`nano /usr/local/etc/matrix-synapse/homeserver.yaml`:
```
enable_registration: true
registration_requires_token: true
```
Save and exit

#### Create registration-bot user on homeserver & install
```
# register_new_matrix_user -c /usr/local/etc/matrix-synapse/homeserver.yaml http://localhost:8008
New user localpart [root]: registration-bot
Password: (enter a strong password here)
Make admin [no]: no

# pip install matrix-registration-bot
# pip install simplematrixbotlib
# mkdir /usr/local/etc/matrix-registration-bot
```
#### Configure
`nano /usr/local/etc/matrix-registration-bot/config.yml`:
```
bot:
  server: "https://synapse.example.com"
  username: "registration-bot"
  access_token: "verysecret"
  # It is also possible to use a password based login by commenting out the access token line and adjusting the line below
  # password: "secretpassword" 
api:
  # API endpoint of the registration tokens
  base_url: 'https://synapse.example.com'
  # Access token of an administrator on the server
  token: "supersecret"
logging:
  level: DEBUG|INFO|ERROR
```
Save and exit

#### Test 
```
# python -m matrix_registration_bot.bot
```
It should work, try it out! Press Ctrl+C to stop when done testing

#### Create system user & permissions
```
# pw adduser mrb -d /nonexistent -s /usr/sbin/nologin -c "matrix-registration-bot"
# chown -R mrb:mrb /usr/local/etc/matrix-registration-bot
```

#### Startup script
`touch /usr/local/etc/rc.d/mrb && chmod +x /usr/local/etc/rc.d/mrb && nano /usr/local/etc/rc.d/mrb`:
```
#!/bin/sh
#
# PROVIDE: mrb
# REQUIRE: synapse
# KEYWORD:

. /etc/rc.subr
name="mrb"
rcvar="mrb_enable"
mrb_command="/usr/local/bin/python -m matrix_registration_bot.bot"
pidfile="/var/run/${name}.pid"
mrb_chdir="/usr/local/etc/matrix-registration-bot"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -u mrb -r -f ${mrb_command}"

load_rc_config $name
: ${mrb_enable:=no}

run_rc_command "$1"
```
Save and exit.

```
# sysrc mrb_enable="YES"
# service mrb start
```

## Future admin panel:
https://github.com/Awesome-Technologies/synapse-admin
