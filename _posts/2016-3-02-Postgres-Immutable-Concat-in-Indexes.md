---
title: 'Postgres Immutable Concat in Indexes'
author: Tyler
layout: post
permalink: http://tylercubell.com/postgres-immutable-concat-in-indexes
categories:
  - Posts
---

The [volitility category](http://www.postgresql.org/docs/8.3/static/xfunc-volatility.html) of Postgres `concat` function is STABLE which means it's `guaranteed to return the same results given the same arguments for all rows within a single statement`. Basically, you're out of luck if you want to use this function in an index. Tom Lane explains in [this post](http://www.postgresql.org/message-id/3361.1410026366@sss.pgh.pa.us):

```
> The pg_catalog.concat() is defined as STABLE function.
> why was STABLE preferred for concat() over IMMUTABLE?

concat() invokes datatype output functions, which are not necessarily
immutable.  An easy example is that timestamptz_out's results depend
on the TimeZone setting.
```

In my case I'm only dealing with integer and text columns so I modified the original function (STABLE -> IMMUTABLE), creating `immutable_concat`.

```sql
CREATE OR REPLACE FUNCTION immutable_concat(VARIADIC "any")
  RETURNS text AS
'text_concat'
  LANGUAGE internal IMMUTABLE
  COST 1;
  ```
