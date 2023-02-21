---
title: Ultimate Vim
tags: [vim]
---

## Readonly mode

In case we need only view (read) file it's better to open it in readonly mode by specifing `-R` parameter:

```console
$ vim -R <file>
```

This is exaclty the same as using `view`:

```console
$ view <file> # will open file in readonly mode
```

Attempt to write file will bring this message:

```
E45: 'readonly' option is set (add ! to override)
```

In readonly mode you still can edit file, but to write change you have to override it explicitly.

To open Vim in readonly and modifiable set to off use `-M` parameter:

```console
$ vim -M <file>
```

Attempt to modify file will bring this message:

```
E21: Cannot make changes, 'modifiable' is off
```

## Sessions



## Navigation

### Go down or up in the jumplist

After you navigated to some other location, for example using `g`, `GG` commands or even next search pattern,
it's possible to return to the previous position ("back" in the jumplist) using `Ctrl-o`.

Where `Ctrl-i`/`Tab` is to go "forward" in the jumplist.

It also works between files, navigation can open previous files with latest position.

> To have `Ctrl-i` working check if `<C-i>` and `<Tab>` are not remapped.
{: .prompt-tip }

### Navigation summary

Up/down in the jumplist: `Ctrl-o`/`Ctrl-i`

