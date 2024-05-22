# requests-unixsocket

<a href="https://badge.fury.io/py/requests-unixsocket2">
    <img src="https://badge.fury.io/py/requests-unixsocket2.svg" alt="Latest Version on PyPI" />
</a>
<a href="https://gitlab.com/thelabnyc/requests-unixsocket/-/commits/master">
    <img alt="pipeline status" src="https://gitlab.com/thelabnyc/requests-unixsocket/badges/master/pipeline.svg" />
</a>
<a href="https://gitlab.com/thelabnyc/requests-unixsocket/-/commits/master">
    <img alt="coverage report" src="https://gitlab.com/thelabnyc/requests-unixsocket/badges/master/coverage.svg" />
</a>
<a href="https://gitlab.com/thelabnyc/requests-unixsocket/-/releases">
    <img alt="Latest Release" src="https://gitlab.com/thelabnyc/requests-unixsocket/-/badges/release.svg" />
</a>

Use [requests](http://docs.python-requests.org/) to talk HTTP via a UNIX domain socket.

## Usage

### Explicit

You can use it by instantiating a special `Session` object:

```py
import json

import requests_unixsocket

session = requests_unixsocket.Session()

r = session.get('http+unix://%2Fvar%2Frun%2Fdocker.sock/info')
registry_config = r.json()['RegistryConfig']
print(json.dumps(registry_config, indent=4))
```


### Implicit (monkeypatching)

Monkeypatching allows you to use the functionality in this module, while making minimal changes to your code. Note that in the above example we had to instantiate a special `requests_unixsocket.Session` object and call the `get` method on that object. Calling `requests.get(url)` (the easiest way to use requests and probably very common), would not work. But we can make it work by doing monkeypatching.

You can monkeypatch globally:

```py
import requests_unixsocket

requests_unixsocket.monkeypatch()

r = requests.get('http+unix://%2Fvar%2Frun%2Fdocker.sock/info')
assert r.status_code == 200
```

or you can do it temporarily using a context manager:

```py
import requests_unixsocket

with requests_unixsocket.monkeypatch():
    r = requests.get('http+unix://%2Fvar%2Frun%2Fdocker.sock/info')
    assert r.status_code == 200
```

### Abstract namespace sockets

To connect to an [abstract namespace socket](https://utcc.utoronto.ca/~cks/space/blog/python/AbstractUnixSocketsAndPeercred) (Linux only), prefix the name with a NULL byte (i.e.: `\0`) - e.g.:

```py
import requests_unixsocket

session = requests_unixsocket.Session()
res = session.get('http+unix://\0test_socket/get')
print(res.text)
```

For an example program that illustrates this, see `examples/abstract_namespace.py` in the git repo. Since abstract namespace sockets are specific to Linux, the program will only work on Linux.

## See also

- https://github.com/msabramo/requests-unixsocket - origin of this project.
- https://github.com/httpie/httpie-unixsocket - a plugin for `HTTPie <https://httpie.org/>`_ that allows you to interact with UNIX domain sockets.
