somark-exec
===========

This script enables `SO_MARK`-unaware applications to run with a socket mark.

While there are alternative methods to accomplish similar results, such as
l3mdev VRF or nftables matching based on socket UID, they require complicated
preparation steps.

This script leverages cgroup2 and an eBPF program attached at the
`BPF_CGROUP_INET_SOCK_CREATE` hook to set the `SO_MARK` socket option to every
socket created by the target application.


Usage
-----

```console
$ somark-exec <mark> <cmd> ...
```

You probably have to run it as root. Loading eBPF program requires
`CAP_SYS_ADMIN` and it is likely your cgroup2 mount is writable only by root.


Examples
--------

curl is an application that does not (AFAIK!) provide command line options for
`SO_MARK`.

```console
$ ip rule
0:      from all lookup local
100:    from all fwmark 0xa lookup 10
32766:  from all lookup main
32767:  from all lookup default
$ curl httpbin.org/ip
{
  "origin": "45.###.###.###"
}
$ sudo ./somark-exec 10 curl httpbin.org/ip
{
  "origin": "220.###.###.###"
}
```


Known issues
------------

This script `exec`s the target application. After exiting it, the cgroup is no
longer necessary. However, it will not be deleted automatically. Generally,
this is benign and it gets cleared upon a reboot, but you may choose to
manually delete it. In typical systems using systemd, you can find it in
`/sys/fs/cgroup/user.slice/user-*.slice/session-*.scope`.

```console
$ sudo rmdir /sys/fs/cgroup/user.slice/user-1000.slice/session-3.scope/somark-exec-100
```


License
-------

somark-exec is licensed under the MIT license. See also COPYING.
