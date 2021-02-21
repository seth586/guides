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


