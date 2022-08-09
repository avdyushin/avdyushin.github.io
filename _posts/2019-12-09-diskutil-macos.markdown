+++
date = 2019-12-09T19:18:02+01:00
title = "Disk Utility on macOS"
description = ""
slug = ""
tags = ["macOS"]
categories = []
externalLink = ""
series = []
+++

Disk Utility in Terminal

List available volumes:

```bash
$ diskutil list
[...]
/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.3 GB    disk2
   1:             Windows_FAT_32 NO NAME                 31.3 GB    disk2s1
```

Format to specific file system:

```bash
$ sudo diskutil eraseDisk FAT32 NO_NAME MBRFormat /dev/disk2
```

Format to EXT3:

```bash
# Install e2fsprogs
$ brew install e2fsprogs
# Unmount is important
$ diskutil unmountDisk /dev/disk2
# Format as EXT3
$ sudo $(brew --prefix e2fsprogs)/sbin/mkfs.ext3 /dev/disk2
```

