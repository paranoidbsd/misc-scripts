available_updates
=================

Simple scripts I use to update my ports tree.

The scripts assume you use `portsnap`, `portmaster` and `pkg`. There
is not much in terms of resiliency or error handling. Use at your own
risk.
Error handling, readability and comments received a small update. It's
still a shellscript you execute as root. Read it.

`available_updates` updates the ports tree and rebuilds the info file,
`recalc_updates` only rebuilds the info file.

It creates an info file `/root/available_updates.txt` with the
following layout:

```
...
graphics/png                        V: 1.5.20       < 1.6.16
graphics/poppler                    P: 0.26.3       < 0.26.3_1
graphics/vigra                      P: 1.10.0_3     < 1.10.0_4
graphics/webp                       P: 0.4.2_1      < 0.4.2_2
lang/gcc                            U: 4.8.3_2      < 4.8.4
lang/php56                          U: 5.6.3        < 5.6.4
mail/thunderbird                    P: 31.3.0       < 31.3.0_1
misc/gnomehier                      O: 3.0          ? orphaned
...
```

It provides an amalgamated view of `pkg version -voUPL=`, `pkg audit`
and `pkg info`. Depending on how you do your updates, it could help
you with your update strategy (does for me, that's why I wrote it).

The line format is `origin`, `flag`, `current version`, `predicate` and
`new version`. The spacing between columns is not static, but tries to
be semi-intelligent based on the longest origin path and longest
current version number.

The flags are:
* `V` currently installed version has a known vulnerability
* `!` version available for update has a known vulnerability
* `P` port revision update
* `U` upstream version update
* `O` orphaned port
* `-` for cases that have no flag of their own
* `X` something went wrong with the script somewhere

Too trivial for copyright.

Currently Known Problems
========================

False Positive ! Flag
---------------------

If a security issue is registered within VuXML as affecting the
following versions:

```
ruby20 < 2.0.0.645,1
ruby   < 2.1.6,1
ruby22 < 2.2.2,1
```

and your `/etc/make.conf` contains the following definition:

```
DEFAULT_VERSIONS=   ruby=2.0
```

at the time that you have built the package (yourself), then
`pkg query %n lang/ruby20` will return `ruby` instead of
`ruby20`. The `recalc_updates` script in turn will use this to
construct `ruby-2.0.0.645,1` as the version available for updating,
issuing `pkg audit 'ruby-2.0.0.645,1'` to determine if the port should
be flagged with `!`. But this will be compared, in this example,
against `ruby < 2.1.6,1` instead of `ruby20-2.0.0.645,1'` and will thus
give a false-positive vulnerability warning.

By using the output of the following command:

```
~% make -D__MAKE_CONF -C /usr/ports/lang/ruby20 package-name
ruby20-2.0.0.645,1
```

instead of assembling it via `pkg query %n` this should be
resolvable, since the settings in `/etc/make.conf` can be ignored this
way.
