Assorted rc.d scripts
=====================


FreeBSD generally mounts filesystems in multiple stages: 

1. `mountcritlocal`, local fstab filesystems
2. `zfs`, zfs automounted filesystems
3. `mountcritremote`, nfs filesystems in fstab
4. `mountlate`

In terms of ordering, `mountcritlocal` and `zfs` are required for the
`FILESYSTEMS` rcorder meta target, meaning they run before scripts run
that expect there to be all the working, in order filesystems.

`mountcritremote` runs after that, so that the mountpaths for the
remote filesystems have been provided. `mountlate` is is the last one,
running after the `DAEMON` meta target, ie. when generic userland
daemons are started.

This poses a problem if you, for example, have your `/var` on ZFS
mounted via `zfs_enable` and would like `/var/run` to be a `tmpfs`. By
the time `mountcritlocal` is called, `/var` will not be mounted yet.
`mountcritremote` is semantically the wrong location, and by the time
`mountlate` is called, `/var/run` will already have been in use.
Mostly by `ldconfig`, making for a very unpleasant experience when you
mount an empty tmpfs over the path lookup information used to find
shared libraries.

The two scripts in this directory, `pbsd_mounttmpfs` and
`pbsd_mountpseudofs` were written to solve this. They depend on
everything `FILESYSTEMS` depends on, but want to be run before it -
they become the last scripts before the `FILESYSTEMS` barrier.

They exploit the fact that kernel provided pseudo filesystems like
`tmpfs`, `procfs`, `fdescfs` and `mqueuefs` will in fact never require
interaction with some sort of running userspace daemon. It is therefor
safe to mount these filesystems much earlier, even if they are marked
late. And during `mountlate` they will be recognized as having already
been mounted.

The scripts are namespaced `pbsd_*` mainly so I recognize them during
`mergemaster` and do not delete them (`mergemaster` will identify them
as stale and ask to delete them).

If you have a genuine need to mount these filesystem types during the
`mountlate` stage, these scripts do not work for you. For that an
additional keyword like `late` would have to be introduced.

pbsd_mounttmpfs
===============

Will mount all filesystems in `/etc/fstab` of type `tmpfs` and marked
`late`. Runs after `mountcritlocal` and `zfs`.

pbsd_mountpseudofs
==================

Will mount all filesystems in `/etc/fstab` of the types `procfs`,
`fdescfs` and `mqueuefs` that are marked `late`. Runs after
`pbsd_mounttmpfs`, so an `mqueuefs` can be mounted below a `tmpfs`.

Note
====

These two scripts are easily folded into one by adding `tmpfs` as the
first type to the for loop in `pbsd_mountpseudofs`; even satisfying
the mount ordering implications.
I have not done this to clearly differentiate between `tmpfs`, a real
filesystem, and the actual pseudo filesystems which I like to think of
as projections into the filesystem namespace.

If you use them, you need to keep track of changes to the `REQUIRE:`
section of `/etc/rc.d/FILESYSTEMS` and update them accordingly.
Otherwise strange things might happen.
As self-contained scripts these are not as volatile as changes to the
rc system usually are, but they still require attention.
