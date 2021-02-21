`pkg install py37-virtualenv py37-pybind11 pybind11 rust`

`virtualenv -p /usr/local/bin/python3.7 .`

`chmod +x ~/bin/activate_this.py`

`source ~/bin/activate.csh`

`pip install --global-option=build_ext --global-option="-I/usr/local/include" --upgrade 'mautrix-signal[all]'`

