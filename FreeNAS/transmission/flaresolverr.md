### FlareSolverr
https://github.com/FlareSolverr/FlareSolverr

```
pkg install chromium python39 py39-pip xorg-vfbserver git-tiny
cd /usr/local/share
git clone https://github.com/FlareSolverr/FlareSolverr.git
cd /usr/local/share/FlareSolverr
python3.9 -m pip install -r requirements.txt
pw adduser lnd -d /home/flaresolverr -s /usr/sbin/nologin
mkdir -p /home/flaresolverr && chown flaresolverr:flaresolverr /home/flaresolverr
mkdir /var/run/flaresolverr && chown flaresolverr:flaresolverr /var/run/flaresolverr
```





`nano /usr/local/etc/rc.d/flaresolverr`:
```
#!/bin/sh
#
# PROVIDE: flaresolverr
# REQUIRE: networking openvpn
# KEYWORD:

. /etc/rc.subr

name="flaresolverr"
rcvar="${name}_enable"
flaresolverr_user="flaresolverr"
flaresolverr_chdir="/usr/local/share/FlareSolverr
load_rc_config ${name}
: ${flaresolverr_enable:="NO"}

pidfile="/var/run/flaresolverr/flaresolverr.pid"

command="/usr/sbin/daemon"
command_args="-u "${flaresolverr_user}" -p ${pidfile} -f /usr/local/bin/python3.9 /usr/local/share/FlareSolverr/src/flaresolverr.py"

run_rc_command "$1"
```

```
chmod +x /usr/local/etc/rc.d/flaresolverr
sysrc flaresolverr_enable=TRUE
service flaresolverr start
```
