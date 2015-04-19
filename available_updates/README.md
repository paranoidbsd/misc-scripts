available_updates
=================

Simple scripts I use to update my ports tree.

The scripts assume you use `portsnap`, `portmaster` and `pkg`.
Originally there was not much in terms of resiliency or error handling.
Error handling, readability and comments received small updates since.
It's still a shellscript you execute as root. Read it and use at your
own risk.

`available_updates` updates the ports tree via `portsnap`, rebuilds
the ports INDEX via `make` and performs some cleanup jobs via
`portmaster`. It then calls `recalc_updates`.
`recalc_updates` creates the info file that list what updates are
available and of which type they are.

The info file is `/root/available_updates.txt` with the
following layout:

```
...
graphics/png          V+ 1.5.20       < 1.6.16
graphics/poppler      P: 0.26.3       < 0.26.3_1
graphics/vigra        P: 1.10.0_3     < 1.10.0_4
graphics/webp         P: 0.4.2_1      < 0.4.2_2
lang/gcc              U: 4.8.3_2      < 4.8.4
lang/php56            U: 5.6.3        < 5.6.4
mail/thunderbird      P+ 31.3.0       < 31.3.0_1
misc/gnomehier        O: 3.0          = orphaned (REMOVED)
misc/foobar           O: 1.0          ? orphaned (MOVED: misc/foobar1)
net-mgmt/snafu        !: 23.1         < 42.0
textproc/lala         D: 1.2.3        > 1.1.13 (DOWNGRADE)
...
```

It provides an amalgamated view of `pkg version -voUL=`, `pkg audit`
and `pkg info`. For orphaned ports, `MOVED` is checked to determine
whether the port was renamed or removed. For all ports, `UPDATING`
is checked for relevant entries.
Depending on how you do your updates, it could help you with your
update strategy (does for me, that's why I wrote it).

As it does not generate new information about your ports, its
"selling points" are:
* condensed information view from multiple sources
* pkg-audit info not only on what is installed, but the available
  update as well to indicate if a fix exists
* easy to differentiate between upstream version changes which are
  more likely to cause lib problems and revision/epoch changes by
  the port maintainer
* additional info for orphaned ports
* indicator if a relevant UPDATING entry exists

The line format is `origin`, `flag:` or `flag+`, `current version`,
`predicate` and `available version`. The spacing between columns is not
static, but tries to be semi-intelligent based on the longest origin path
and longest current version number.

The flags are:
* `V` currently installed version has a known vulnerability
* `!` version available for update has a known vulnerability
* `P` port revision or epoch update
* `U` upstream version update
* `O` orphaned port
* `D` port provides a downgrade of currently installed version
* `-` for cases that have no flag of their own
* `X` something went wrong with the script somewhere

If the flag is followed by a `+` instead of a `:`, then at least one
entry in UPDATING exists for which the following is true:
* the port's origin is listed in the entry's AFFECTS line
* the entry is newer than the current package's install date

The predicates are:
* `<` for available updates
* `<` for available downgrades
* `=` for orphaned ports that have been removed
* `?` for orphaned ports that have been moved/replaced

Too trivial for copyright.
