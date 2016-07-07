---
title: 'Postgres Run Query on Multiple Tables'
author: Tyler
layout: post
permalink: /postgres-run-query-on-multiple-tables
categories:
  - Posts
---

If a query needs to be run on multiple tables with similar schemas, use this [anonymous code block](https://www.postgresql.org/docs/9.5/static/sql-do.html) to [reduce repetition](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

```sql
DO $$
DECLARE
  current_table text;
BEGIN
    FOREACH current_table IN ARRAY ARRAY['table1', 'table2']
    LOOP
        EXECUTE format('DELETE FROM %I WHERE foo = %L', current_table, 'bar');
    END LOOP;
END $$;
```
