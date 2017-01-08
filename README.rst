runit-user-session
==================

This service will start a runsvdir process for a given user that will manage runit services in::

    $HOME/.local/service

requirements
------------

* libcgroup-utils - The session uses cgroups to limit available pids.

user-sessions
-------------

create a user session service::

    $ cp -r user-session /etc/sv/user."$USER"
    $ ln -s /etc/sv/user."$USER" /var/service/user."$USER"

and services in *~/.local/service*. Runit should now automatically run all services.
To control the services, use the provided usv script or add a function to your shell rc::

    usv () {
        SVDIR=~/.local/service sv $@
    }

now run (assuming you have the user-session running)::

    $ usv start aria2c
    ok: run: aria2c: (pid 12237) 1061s
    $ usv status aria2c
    run: aria2c: (pid 12237) 1353s
    $ usv stop aria2c
    ok: down: aria2c: 1s, normally up

Stopping the user-session should also stop all child services::

    $ usv status aria2c
    ok: run: aria2c: (pid 12237) 1061s
    $ ps 14335
      PID TTY      STAT   TIME COMMAND
    12237 ?        S      0:00 aria2c --enable-rpc=true --disable-ipv6 --check-certificate=false
    $ sv user-session stop
    ok: down: user-session: 0s, normally up
    $ ps aux | grep aria2c
    admin  14458  0.0  0.0  10756  2192 pts/6    S+   00:12   0:00 grep aria2c

