---
title: Productivity with Zsh
date: "2022-07-06 16:57:52 +0200"
category: Productivity
tags: [zsh]
---

## Navigation
### Directories history
#### Quick return to the previous directory

Suppose you are inside some directory and need to temporary switch to another one.
Then to return back to the previous directory simply type:

```console
$ cd -
```

Or using environment variable `$OLDPWD` if such behaviour needed in shell script, for example

```console
$ cd $OLDPWD
```

And `$PWD` variable for current directory path, just in case.

#### Keep directories of interest inside build in stack

Using built-in commands `pushd` and `popd` it's possible to navigate over file system relatively quickly.

```console
$ pushd /my/path
```
this will add `/my/path`{: .filepath } on the stack, and changes to a new directory.

```console
$ popd
```

this will pop a directory from the stack and change to it.

To see history list use `dirs` command:

```console
$ dirs
$ dirs -l # full paths
$ dirs -v # as a list
$ dirs -c # clean
```

__NOTE__: directory stack always contains `$PWD` as the first element. Think of `cd` command which changes `$PWD` variable.

Navigate over history:

```console
$ cd +3
```

Configuration using options:

```shell
DIRSTACKSIZE=5
setopt autopushd # call pushd on each directory change
setopt pushdsilent # don't print directories history
setopt pushdignoredups # ignore copies
setopt pushdminus # exchange + and -, e.g cd +3 will be cd -3
```

## Files

### Moving around

Move files in `zsh` with `zmv` command.

First of all enable it in yours `~/.zshenv`{: .filepath } (disabled by default):

```console
$ echo 'autoload zmv' >> ~/.zshenv
$ source ~/.zshenv
```

As example rename all `*.md` files into `*.markdown`:

```console
$ zmv -W '*.md' '*.markdown'
$ noglob zmv -W *.md *.markdown
```

With `noglob` no need to quote arguments.

## Arguments

### Reuse arguments from previous command

```console
$ echo 1 2 3 4
```

* `!$` — the last argument `4`
* `!*` — all the arguments `1 2 3 4`

### Repeat last command

`!!` to repeat last used command, for example:

```console
$ echo 1 2 3
$ sudo !!
```

## Links

- [Changing Directories](https://zsh.sourceforge.io/Doc/Release/Options.html#Changing-Directories)

