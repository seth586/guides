`pkg install py37-virtualenv olm rust py37-pillow`

`virtualenv -p /usr/local/bin/python3.7 .`

`source ~/bin/activate.csh`

`pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade python-olm`

`pip install --upgrade 'mautrix-signal[all]'`

