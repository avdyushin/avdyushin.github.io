---
title: Using Grep to find files
date: 2019-11-20T19:28:27+01:00
category: Productivity
tags: [grep, vim]
---

The `grep` utility can be used to find files contains occurrences of the given pattern.

```console
$ man grep
```

### Add colors

There is an option to have matching text to be marked up. It's nice to have it each time automatically.
So it can be done by defining alias:

```shell
alias grep='grep --color'
```

### Find files

To find all files contains occurrences of the pattern 'vi[m]?' at the end of a line:

```console
$ grep -rEi 'vi[m]?$' .
```

Where:

* `-r` — recursively
* `-E` — extended regular expression
* `-i` — case insensitive matching

To print line numbers and show context surrounding match:

```console
$ grep -rEin -C1 'vim$' .
--
./vim-clipboard.md-12-
./vim-clipboard.md:13:vim
./vim-clipboard.md-14-:h registers
--
--
./vim-clipboard.md-35-bash
./vim-clipboard.md:36:$ brew install vim
./vim-clipboard.md-37-
--
```

Where:

* `-n` — print line number
* `-C` — show context

To print only filenames:

```console
$ grep -rl 'vim$' .
./vim-clipboard.md
```

> Duplicates were removed

Where:

* `-l` — show only names of files

To find all files *not* containing string:

```console
$ grep -rL 'vim' .
```

Where:

* `-L` — show only names of files not containing string

More useful options:

* `-I` — ignore binary files

### Combine commands

It's possible to combine results and provide them into other tools using pipes.

For example to execute `rm` command for each file in search results:

```console
$ grep -r 'string' . | xargs rm
```

> This will remove all files contained 'string' pattern in current directory.
{: .prompt-danger }

> If file name contains spaces, here is a trick for macOS:
>
> `grep -r 'string' . | tr '\n' '\0' | xargs -0 rm`
{: .prompt-tip }

To open result files in Vim:

```bash
$ vim $(grep -rL 'pattern' .)
```

Inside vim use `:bn` command to open next buffer and `:bp` command for previous.

### Summary

| Option | Action                                             |
| :----- | :------------------------------------------------- |
|  `-r`  | Recursively search subdirectories                  |
|  `-E`  | Extended regular expression                        |
|  `-i`  | Case insensitive matching                          |
|  `-w`  | Match whole word                                   |
|  `-l`  | Show only names of files                           |
|  `-I`  | Ignore binary files                                |
|  `-n`  | Print line number                                  |
|  `-C`  | Show context                                       |
|  `-L`  | Show only names of files **not** containing string |
