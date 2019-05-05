[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]

### Electrs: Electrum In Rust

Read up more on electrs at its github page [here](https://github.com/romanz/electrs)


Download prerequisites:
```
# pkg install rust git
```
Electrs uses some new features of rust (v1.34), so if pkg says it will install a version lower than 1.34, change our pkg repository from quarterly releases to latest releases:
```
# mkdir -p /usr/local/etc/pkg/repos
# nano /usr/local/etc/pkg/repos/FreeBSD.conf
FreeBSD: {
    url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
# pkg install rust
```

Lets compile!

```
# cd ~
# git clone https://github.com/romanz/electrs
# cd electrs
# cargo build --release
```
You will get an error. Find the error that looks like this:
```
/root/.cargo/registry/src/github.com-1ecc6299db9ec823/sysconf-0.3.4/src/raw.rs
```
The path may vary as the author updates his program. Use nano to edit that file, and replace the following lines:
```
ScXbs5Ilp32Off32 = sc!(_SC_XBS5_ILP32_OFF32),
ScXbs5Ilp32Offbig = sc!(_SC_XBS5_ILP32_OFFBIG),
ScXbs5LpbigOffbig = sc!(_SC_XBS5_LPBIG_OFFBIG),
```
with:
```
ScXbs5Ilp32Off32 = sc!(_SC_V6_ILP32_OFF32),
ScXbs5Ilp32Offbig = sc!(_SC_V6_ILP32_OFFBIG),
ScXbs5LpbigOffbig = sc!(_SC_V6_LPBIG_OFFBIG),
```

Now try to compile, it should suceed:
```
# cargo build --release
# cp /root/electrs/target/release/electrs /usr/local/bin
# mkdir /var/db/electrs
# electrs -vvv --db-dir=/var/db/electrs --electrum-rpc-addr=192.168.84.208:50002 --daemon-dir=/var/db/bitcoin
```
Make sure to replace `electrum-rpc-addr=` with your bitcoin jail's IP to serve local connections, or use 127.0.0.1:50002 if you plan on serving remote connections over tor authenticated hidden services.

Electrs will now index the blockchain into its own database. This can take a few hours, depending on your CPU and disk IO. You may get spammed by a `WARN - failed to export stats: failed to read stats`, just ignore it. When its done indexing, it will start to serve connections.

Right click on your windows electrum client, select properties, and modify the shortcut to resemble below:
```
"C:\Program Files (x86)\Electrum\electrum-3.3.4.exe" -1 -s 192.168.84.208:50002:t
```
Start up electrum client. It should connect! Terminate electrs with ctrl+c. Verify it is no longer running with `ps aux`. If it is still running, kill it with `kill -9 <pid>`, whereas `<pid>` is the number under the PID column from command `ps aux`. 

Now lets write an rc.d script so it automatically starts as a service:

```
# nano /usr/local/etc/rc.d/electrs
```

Paste the following script:
```
#!/bin/sh
#
# PROVIDE: electrs
# REQUIRE: bitcoind
# KEYWORD:

. /etc/rc.subr

name="electrs"
rcvar="electrs_enable"
electrs_command="/usr/local/bin/electrs --db-dir=/var/db/electrs --electrum-rpc-addr=192.168.84.208:50002 --daemon-dir=/var/db/bitcoin"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${electrs_command}"

load_rc_config $name
: ${electrs_enable:=no}

run_rc_command "$1"
```
Save, (ctrl+o,enter) and exit (ctrl+x)

Make the script executable:
```
# chmod +x /usr/local/etc/rc.d/electrs
```
And enable on startup:
```
# sysrc electrs_enable="YES"
```
Give it a whir:
```
# service electrs start
```
[ [<< Back to Extras](https://github.com/seth586/guides/blob/master/FreeNAS/extras.md) ]
