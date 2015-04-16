recompile_system
================

Lazy people wrapper around the following main command sequence:

```
zfs snap -r tank@${svn_revision}
make -C /usr/src clean
make -C /usr/src cleandir
make -C /usr/src buildworld
make -C /usr/src buildkernel
```

It does however automatically set up convenient logging. The root of
the log hierarchy is `/root/compile.logs`. Every run creates a
subdirectory of the format `${unix_timestamp}_r${svn_revision}`.
Within this directory the following backup files will be created (and
xz compressed):
* `${svn_revision}_buildkernel.log`, script log of the buildkernel run
* `${svn_revision}_buildworld.log`, script log of the buildworld run
* `generic.conf`, a copy of the GENERIC kernel configuration from the
  tree at the time of the run. This is mostly so you can diff between
  them to detect changes to GENERIC that you need to carry over to a
  custom kernel config (alternative to `svn diff ...`)
* `kernel.conf`, a copy of the kernel configuration that was used.
  This may be a second copy of GENERIC or whatever make.conf points at
* `make.conf`, a copy of `/etc/make.conf`
* `progress`, a small log file with timestamped lines for cleanup,
  buildworld, buildkernel and finish
* `src.conf`, a copy of `/etc/src.conf`

The result is a hierarchy like this:

```
/root/compile.logs
├── 1428755539_r281435
│   ├── 281435_buildkernel.log.xz
│   ├── 281435_buildworld.log.xz
│   ├── generic.conf.xz
│   ├── kernel.conf.xz
│   ├── make.conf.xz
│   ├── progress.xz
│   └── src.conf.xz
└── 1429034928_r281531
    ├── 281531_buildkernel.log.xz
    ├── 281531_buildworld.log.xz
    ├── generic.conf.xz
    ├── kernel.conf.xz
    ├── make.conf.xz
    ├── progress.xz
    └── src.conf.xz
```

The script has embedded configuration, for example the pool/dataset
name for snapshotting needs to be adjusted etc.yadda.yadda.

Disclaimer
==========

Usual disclaimer about internet-downloaded scripts executed as root.
Slippery. wet.
