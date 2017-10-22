pystemd
=======

This library allows you to talk to systemd over dbus from python, without
actually thinking that you are talking to systemd over dbus. This allows you to
programmatically start/stop/restart/kill and verify services status from
systemd point of view, avoiding executing `subprocess.Popen(['systemctl', ...`
and then parsing the output to know the result.


Show don't tell
---------------

In software as in screenwriting, its better to show how things work instead of
tell. So this is how you would use the library from a interactive shell.  

    In [1]: from pystemd.systemd1 import Unit
    In [2]: unit = Unit(b'postfix.service')
    In [3]: unit.load()

Note: you need to call `unit.load()` because by default `Unit` will not load the
unit information as that would require do some IO. You can auto load the unit by
`Unit(b'postfix.service', _autoload=True)`

Once the unit is loaded, you can interact with it, you can do by accessing its
systemd's interfaces:

    In [4]: unit.Unit.ActiveState
    Out[4]: b'active'

    In [5]: unit.Unit.StopWhenUnneeded
    Out[5]: False

    In [6]: unit.Unit.Stop(b'replace') # require privilege account
    Out[6]: b'/org/freedesktop/systemd1/job/6601531'

    In [7]: unit.Unit.ActiveState
    Out[7]: b'inactive'

    In [8]: unit.Unit.Start(b'replace') # require privilege account
    Out[8]: b'/org/freedesktop/systemd1/job/6601532'

    In [9]: unit.Unit.ActiveState
    Out[9]: b'active'

    In [10]: unit.Service.GetProcesses()
    Out[10]:
    [(b'/system.slice/postfix.service',
        1754222,
        b'/usr/libexec/postfix/master -w'),
     (b'/system.slice/postfix.service', 1754224, b'pickup -l -t fifo -u'),
     (b'/system.slice/postfix.service', 1754225, b'qmgr -l -t fifo -u')]

    In [11]: unit.Service.MainPID
    Out[11]: 1754222

The `systemd1.Unit` class provides shortcuts for the interfaces in the systemd
namespace, as you se above, we have  Service (org.freedesktop.systemd1.Service)
and Unit (org.freedesktop.systemd1.Unit). Others can be found in
`unit._interfaces` as:

```
In [12]: unit._interfaces
Out[12]:
{'org.freedesktop.DBus.Introspectable': <org.freedesktop.DBus.Introspectable of /org/freedesktop/systemd1/unit/postfix_2eservice>,
 'org.freedesktop.DBus.Peer': <org.freedesktop.DBus.Peer of /org/freedesktop/systemd1/unit/postfix_2eservice>,
 'org.freedesktop.DBus.Properties': <org.freedesktop.DBus.Properties of /org/freedesktop/systemd1/unit/postfix_2eservice>,
 'org.freedesktop.systemd1.Service': <org.freedesktop.systemd1.Service of /org/freedesktop/systemd1/unit/postfix_2eservice>,
 'org.freedesktop.systemd1.Unit': <org.freedesktop.systemd1.Unit of /org/freedesktop/systemd1/unit/postfix_2eservice>}

 In [13]: unit.Service
 Out[13]: <org.freedesktop.systemd1.Service of /org/freedesktop/systemd1/unit/postfix_2eservice>
```

Each interface has methods and properties, that can access directly as
`unit.Service.MainPID`, the list of properties and methods is in `.properties`
and `.methods` of each interface.

Alongside the `systemd1.Unit`, we also have a `systemd1.Manager`, that allows
you to interact with systemd manager.


```
In [14]: from pystemd.systemd1 import Manager

In [15]: manager = Manager()

In [16]: manager.load()

In [17]: manager.Manager.ListUnitFiles()
Out[17]:
...
(b'/usr/lib/systemd/system/rhel-domainname.service', b'disabled'),
 (b'/usr/lib/systemd/system/fstrim.timer', b'disabled'),
 (b'/usr/lib/systemd/system/getty.target', b'static'),
 (b'/usr/lib/systemd/system/systemd-user-sessions.service', b'static'),
...

In [18]: manager.Manager.Architecture
Out[18]: b'x86-64'

In [19]: manager.Manager.Virtualization
Out[19]: b'kvm'

```


Install
-------

So you like what you see, time to install it. you need to have:

  * Python headers: Just use your distro's package (e.g. python-dev).
  * Six library: for python 2 and 3 compatibility.
  * systemd headers: Chances are you already have this, normally is called
  `libsystemd-dev` on or `systemd-devel`.
  * Cython: Use your distro's package, pip install it or use the official
  installation guide on cython homepage
   http://cython.readthedocs.io/en/latest/src/quickstart/install.html.

then:

    pip install -r requirements.txt # get six
    python setup.py install

License
-------

pystemd is BSD-licensed. We also provide an additional patent grant.
