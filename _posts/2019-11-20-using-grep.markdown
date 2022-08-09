---
date: 2019-11-20T19:28:27+01:00
title: Using Grep to find files
tags: [grep, vim]
---

The grep utility can be used to find files contains occurrences of the given pattern.

```bash
$ man grep
```

### Add colors

```bash
alias grep='grep --color'
```

### Find files

To find all files contains occurrences of the pattern 'vi[m]?' at the end of a line:

```bash
$ grep -rEi 'vi[m]?$' .
```

* `-r` — recursively
* `-E` — extended regular expression
* `-i` — case insensitive matching

To print line numbers and show context surrounding match:

```bash
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

* `-n` — print line number
* `-C` — show context

To print only filenames:

```bash
$ grep -rl 'vim$' .
./vim-clipboard.md
```

> Duplicates were removed

* `-l` — show only names of files

More useful options:

* `-I` — ignore binary files

To find all files *not* containing string:

```bash
$ grep -rL 'vim' .
```

* `-L` - show only names of files not containing string

### Bonus

To execute command for each file in search results:

```bash
$ grep -r 'string' . | xargs rm
```

If file names contains spaces, here is a trick for macOS:

```bash
$ grep -r 'string' . | tr '\n' '\0' | xargs -0 rm
```

To open files in Vim:

```bash
$ vim $(grep -rL 'pattern' .)
```

Inside vim `:bn` to open next buffer and `:bp` previous.

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
