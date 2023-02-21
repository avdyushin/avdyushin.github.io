---
title: Mastering hexdump tool
date: "08-11-2022 16:57:52 +0200"
tags: [hexdump]
---

The `hexdump` is a tool to display files in specific format.
Which is especially handy to work with binary (think of firmware, memory dump, etc) data.

How it looks by default? Let say we have some file:

```console
$ hexdump hello.txt
0000000 6548 6c6c 2c6f 6820 7865 7564 706d 0a21
0000010
```
Not as expected, but to see output in more canonical way we have to specify `-C` flag:

```console
$ hexdump -C hello.txt
00000000  48 65 6c 6c 6f 2c 20 68  65 78 64 75 6d 70 21 0a  |Hello, hexdump!.|
00000010
```

Now we have hexadecimal bytes in two columns followed by ASCII bytes as well.

It's possible to use custom format as well, for example:

```console
 $ hexdump -e '8/1 " %02x" "\n"' hello.txt
 48 65 6c 6c 6f 2c 20 68
 65 78 64 75 6d 70 21 0a
```

Where `'8/1 " %02x" "\n"'` means: 8 bytes per row, formatted as hexadecimal.

More examples:

```console
 $ hexdump -e '16/1 " 0x%02x," "\n"' hello.txt
 0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x20, 0x68, 0x65, 0x78, 0x64, 0x75, 0x6d, 0x70, 0x21, 0x0a,
 ```
