## Groupme

Refer to official documentation: https://gitlab.com/robintown/mx-puppet-groupme

```
git clone https://gitlab.com/robintown/mx-puppet-groupme

cd mx-puppet-groupme

npm install groupme

cp sample.config.yaml config.yaml

nano config.yaml

npm run start -- --register
```

`touch /usr/local/etc/rc.d/groupme && chmox +x /usr/local/etc/rc.d/groupme && nano /usr/local/etc/rc.d/groupme`:
```
#!/bin/sh
#
# PROVIDE: groupme
# REQUIRE:
# KEYWORD:

. /etc/rc.subr
name="groupme"
rcvar="groupme_enable"
groupme_command="/usr/local/bin/node /root/mx-puppet-groupme/build/index.js --config=/root/mx-puppet-groupme/config.yaml --registration-file=/root/mx-puppet-groupme/groupme-registration.yaml"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${groupme_command}"

load_rc_config $name
: ${groupme_enable:=no}

run_rc_command "$1"
```
