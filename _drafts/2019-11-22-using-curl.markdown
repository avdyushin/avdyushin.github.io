---
date: 2019-11-22T11:23:16+01:00
title: "Using cURL"
tags: ["curl"]
---

To sumbit form fields with cURL use `-d` command:

```bash
curl -d 'title=foo&body=bar' $TARGET
```

To set cookie use `-b` command:

```bash
curl -b 'session=foo' $TARGET
```
