[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [**Bitcoin**] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [RTL](freenas_6_rtl.md) ] - [ [Joule](freenas_7_joule.md) ]

### Guide to ₿itcoin & ⚡Lightning️⚡ on 🦈FreeNAS🦈

### Bitcoin Core Install

Secure Socket Shell into your bitcoin jail. SSH is a way to manage your server remotely over a network. When you don’t plug in a monitor & keyboard directly into the server, it’s called a ‘headless’ server. The most popular SSH client is called [PuTTY, download it here](https://www.putty.org/). Connect to your FreeNAS’ IP address, and log in with your root credentials.

```
# iocage list
```

You should see your bitcoin jail listed. Lets switch our console from our base system to our jail.

```
# iocage console bitcoin
```

You're in! Lets start installing stuff!

```
# pkg install autoconf automake boost-libs git gmake libevent libtool libzmq4 openssl pkgconf wget nano
```

Goto Go to https://github.com/bitcoin/bitcoin/releases, find the tar.gz release we want to install. The latest release is 0.17.1 at https://github.com/bitcoin/bitcoin/archive/v0.17.1.tar.gz . Copy the link.

```
# cd ~
# wget https://github.com/bitcoin/bitcoin/archive/v0.17.1.tar.gz
# tar xzvf v0.17.1.tar.gz
```

To see what is in the current directory, type `ls`

You can see our downloaded file, `v0.17.1.tar.gz` and the unpacked directory `bitcoin-0.17.1`, lets change to that bitcoin directory and configure for compiling.
```
# cd bitcoin-0.17.1
# sh 
# ./contrib/install_db4.sh `pwd`
# export BDB_PREFIX='/root/bitcoin-0.17.1/db4'
# ./autogen.sh
# ./configure MAKE=gmake BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include" --without-gui --without-miniupnpc
# gmake check
# gmake install
# tcsh
```
This process may take a while. Once its done and installed, we need to add a rc.d script to automatically start the bitcoin daemon on start. Read more about FreeBSD rc.d scripting [here](https://www.freebsd.org/doc/en_US.ISO8859-1/articles/rc-scripting/index.html).

```
# nano /etc/rc.d/bitcoind
```

Copy the script below and paste the following startup script to nano by right clicking in the terminal.

```
#!/bin/sh
# $FreeBSD$

# PROVIDE: bitcoind
# REQUIRE: LOGIN cleanvar
# KEYWORD: shutdown

#
# Add the following lines to /etc/rc.conf to enable :
# bitcoind_enable (bool):	Set to "NO" by default.
#				Set it to "YES" to enable bitcoind
# bitcoind_user (str)		Set to "bitcoin" by default.
# bitcoind_group (str)		Set to "bitcoin" by default.
# bitcoind_conf (str)		Set to "%%PREFIX%%/etc/bitcoind.conf" by default.
# bitcoind_data_dir (str)	Set to "/root/.bitcoin" by default.
# bitcoindlimits_enable (bool)	Set to "NO" by default.
#				Set it to "YES" to enable bitcoindlimits
# bitcoindlimits_args		Set to "-e -U ${bitcoind_user}" by default


. /etc/rc.subr

name="bitcoind"
rcvar=bitcoind_enable

start_precmd="bitcoind_precmd"
start_cmd="bitcoind_start"
restart_precmd="bitcoind_checkconfig"
reload_precmd="bitcoind_checkconfig"
configtest_cmd="bitcoind_checkconfig"
status_cmd="bitcoind_status"
stop_cmd="bitcoind_stop"
stop_postcmd="bitcoind_wait"
command="/usr/local/bin/bitcoind"
daemon_command="/usr/sbin/daemon"
#pidfile="/var/run/${name}.pid"
extra_commands="configtest"


: ${bitcoind_enable:="NO"}
: ${bitcoindlimits_enable:="NO"}

load_rc_config ${name}

: ${bitcoind_user:="bitcoin"}
: ${bitcoind_group:="bitcoin"}
: ${bitcoind_data_dir:="/root/.bitcoin"}
: ${bitcoind_config_file:="/root/.bitcoin/bitcoin.conf"}
: ${bitcoindlimits_args:="-e -U ${bitcoind_user}"}

# set up dependant variables
procname="${command}"
pidfile="${bitcoind_data_dir}/bitcoind.pid"
required_files="${bitcoind_config_file}"


bitcoind_checkconfig()
{
  echo "Performing sanity check on bitcoind configuration:"
  if [ ! -d "${bitcoind_data_dir}" ]
  then
    echo "Missing data directory: ${bitcoind_data_dir}"
    exit 1
  fi
  chown -R "${bitcoind_user}:${bitcoind_group}" "${bitcoind_data_dir}"

  if [ ! -f "${bitcoind_config_file}" ]
  then
    echo "Missing configuration file: ${bitcoind_config_file}"
    exit 1
  fi
  if [ ! -x "${command}" ]
  then
    echo "Missing executable: ${command}"
    exit 1
  fi
  return 0
}

bitcoind_cleanup()
{
  rm -f "${pidfile}"
}

bitcoind_precmd()
{
  bitcoind_checkconfig

  pid=$(check_pidfile "${pidfile}" "${procname}")
  if [ -z "${pid}" ]
  then
    echo "Bitcoind is not running"
    rm -f "${pidfile}"
  fi

  if checkyesno bitcoindlimits_enable
  then
    eval $(/usr/bin/limits ${bitcoindlimits_args}) 2>/dev/null
  else
    return 0
  fi
}

bitcoind_status()
{
  local pid
  pid=$(check_pidfile "${pidfile}" "${procname}")
  if [ -z "${pid}" ]
  then
    echo "Bitcoind is not running"
    return 1
  else
    echo "Bitcoind running, pid: ${pid}"
  fi
}

bitcoind_start()
{
  echo "Starting bitcoind:"
  cd "${bitcoind_data_dir}" || return 1
  ${daemon_command} -u "${bitcoind_user}" -p "${pidfile}" -f \
    ${command} \
    -conf="${bitcoind_config_file}" \
    -datadir="${bitcoind_data_dir}"
}

bitcoind_stop()
{
  echo "Stopping bitcoind:"
  pid=$(check_pidfile "${pidfile}" "${procname}")
  if [ -z "${pid}" ]
  then
    echo "Bitcoind is not running"
    return 1
  else
    kill ${pid}
  fi
}

bitcoind_wait()
{
  local n=60
  echo "Waiting for bitcoind shutdown:"
  while :
  do
    printf '.'
    pid=$(check_pidfile "${pidfile}" "${procname}")
    if [ -z "${pid}" ]
    then
      printf '\n'
      break
    fi
    sleep 1
    n=$((${n} - 1))
    if [ ${n} -eq 0 -a -f "${pidfile}" ]
    then
      printf "\nForce shutdown"
      kill -9 $(cat "${pidfile}")
      for n in 1 2 3
      do
        printf '.'
        sleep 1
      done
      printf '\n'
      break
    fi
  done
  rm -f "${pidfile}"
  echo "Shutdown complete"
}

run_rc_command "$1"
```
Save the file by pressing CTRL+O, then ENTER. To exit nano, press, CTRL+X

Lets make the startup script executable:
```
# chmod +x /etc/rc.d/bitcoind
```
Lets enable our startup script in /etc/rc.conf
```
# nano /etc/rc.conf
```
Add the following line:
```
bitcoind_enable=yes
```
Save (CTRL+O), ENTER, then exit (CTRL+X)

Lets make bitcoin's [configuration file](https://jlopp.github.io/bitcoin-core-config-generator/):
```
nano /root/.bitcoin/bitcoin.conf
```
Add the following lines:
```
server=1
txindex=1
```
