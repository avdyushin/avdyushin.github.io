---
title: "Vim + macOS + Clipboard"
date: 2019-11-19T20:28:21+01:00
tags: ["vim"]
---

## Using system clipboard in Vim on macOS

When you do copy, cut and paste text in Vim it goes into Vim's own buffer (register).
Actually it has many registers, but we are're interested only in primary (`*`, `unnamed`).

More details on registers in Vim:

```vim
:h registers
```

But it's possible to make it work with system clipboard too.

<!-- more -->

### Pre requirements

Vim requires the `+clipboard` feature flag to be set during compile.
Let's check if Vim has this feature:

```bash
$ vim --version | grep clipboard
+clipboard         +keymap            +printer           +vertsplit
+emacs_tags        -mouse_gpm         -sun_workshop      -xterm_clipboard
```

By default macOS Catalina ships without clipboard. But the good news
we can easy install Vim with the `+clipboard` feature from Homebrew for example:

```bash
$ brew install vim
```

### Setup

We can set default register to `unnamed` clipboard with command:

```vim
set clipboard=unnamed
```

More details on clipboard in Vim:

```vim
:h clipboard
```

I use shared `.vimrc` configuration file across different machines,
so first check if it's supported by Vim:

```vim
" .vimrc
"
" Set default clipboard register to system's unnamed
"
if has('clipboard')
    set clipboard=unnamed
endif
```

Now commands like `y` and `d` will put text into system clipboard
as well as `p` command will take text from clipboard.

### Usage

| Command  | Action                    |
| :------- | :------------------------ |
| `yw`     | Copy word                 |
| `yy`     | Copy line                 |
| `gg"*yG` | Copy entire file          |
| `y$`     | Copy till end of the line |
| `dw`     | Cut word                  |
| `dd`     | Cut line                  |
| `d$`     | Cut till end of the line  |
| `P`      | Paste before cursor       |
| `p`      | Paste after cursor        |
