# Setup python

## Install "pip"

```sh
curl -fSsLo d:\devel\python\get-pip.py https://bootstrap.pypa.io/get-pip.py

python d:\devel\python\get-pip.py
```

## Configure

Edit `python310._pth`

```text
python310.zip
.
Lib\site-packages

# uncomment to run site.main() automatically
import site
```

## Install packages

```sh
pip3 install --upgrade pynvim
pip3 install --upgrade msgpack
```
