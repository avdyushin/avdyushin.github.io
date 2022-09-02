---
title: Using cURL
date: 2019-11-22T11:23:16+01:00
tags: [curl]
---

To sumbit form fields with `curl` use `-d` command:

```console
$ curl -d 'title=foo&body=bar' $TARGET
```

To set cookie use `-b` command:

```console
$ curl -b 'session=foo' $TARGET
```
