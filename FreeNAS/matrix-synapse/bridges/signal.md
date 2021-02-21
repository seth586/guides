Install

```
pkg install py37-virtualenv olm rust py37-pillow nano

virtualenv -p /usr/local/bin/python3.7 .

source ~/bin/activate.csh

pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade python-olm

pip install --upgrade 'mautrix-signal[all]'

cp example-config.yaml config.yaml


```

Create new database

```
sudo -i -u postgres
psql
postgres=# CREATE USER "mautrix-signal" WITH PASSWORD 'password';
\q
exit
```
Configure
nano config.yaml
```
mkdir /usr/local/etc/mautrix-signal
mv config.yaml /usr/local/etc/mautrix-signal/config.yaml
python -m mautrix_signal -g -c /usr/local/etc/mautrix-signal/config.yaml -r /usr/local/etc/mautrix-signal/registration.yaml




```
nano ~/.synapse/homeserver.yaml
app_service_config_files:
  - /usr/local/etc/mautrix-signal/registration.yaml

```
python -m mautrix_signal -c /usr/local/etc/mautrix-signal/config.yaml -r /usr/local/etc/mautrix-signal/registration.yaml
```
