---
title: "Unused Indexes"
categories: ["Examples", "Placeholders"]
tags: ["database", "AWS", "AWS-RDS"] 
date: 2021-11-08T11:32:21+01:00
description: >
  Find out indexes that you _don't_ need.
draft: true
---

It's useful to check the usage of indexes because you can spot some indexes that have never been used and you can considerer deleting them as they are consuming resources.

```sql
select relname as table_name, indexrelname as index_name, idx_scan as times_used 
from pg_catalog.pg_stat_user_indexes 
order by idx_scan desc, relname;
```

But we still can get more information, such as the size of the table or the size of the index with this query: 

```postgres
SELECT
    t.tablename,
    c.reltuples::bigint                            AS num_rows,
    pg_size_pretty(pg_relation_size(c.oid))        AS table_size,
    psai.indexrelname                              AS index_name,
    pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size,
    CASE WHEN i.indisunique THEN 'Y' ELSE 'N' END  AS "unique",
    psai.idx_scan                                  AS number_of_scans,
    psai.idx_tup_read                              AS tuples_read,
    psai.idx_tup_fetch                             AS tuples_fetched,
    idx.indexDef                                   AS index_definition
FROM
    pg_tables t
    LEFT JOIN pg_class c ON t.tablename = c.relname
    LEFT JOIN pg_index i ON c.oid = i.indrelid
    LEFT JOIN pg_stat_all_indexes psai ON i.indexrelid = psai.indexrelid
    LEFT JOIN pg_indexes idx on idx.indexname = psai.indexrelname
WHERE
    t.schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY tablename, number_of_scans desc, pg_relation_size(i.indexrelid) desc;


```

----

And if we are using AWS RDS we could export this data into S3 (see [AWS export query]({{< ref "export-query" >}})) so we can analyze the results quietly and check if we have to remove some indexes:

```postgres
SELECT * FROM aws_s3.query_export_to_s3(
    'SELECT
    t.tablename,
    c.reltuples::bigint                            AS num_rows,
    pg_size_pretty(pg_relation_size(c.oid))        AS table_size,
    psai.indexrelname                              AS index_name,
    pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size,
    CASE WHEN i.indisunique THEN ''Y'' ELSE ''N'' END  AS "unique",
    psai.idx_scan                                  AS number_of_scans,
    psai.idx_tup_read                              AS tuples_read,
    psai.idx_tup_fetch                             AS tuples_fetched
FROM
    pg_tables t
    LEFT JOIN pg_class c ON t.tablename = c.relname
    LEFT JOIN pg_index i ON c.oid = i.indrelid
    LEFT JOIN pg_stat_all_indexes psai ON i.indexrelid = psai.indexrelid
WHERE
    t.schemaname NOT IN (''pg_catalog'', ''information_schema'')
ORDER BY tablename, number_of_scans desc, index_size desc;', 
    'piksel-paletteprod-eu-west-1-metaflow-rds',
    'metadata/rds/blue/metadata_db-indexes-usage.txt',
    'eu-west-1'
);
```

----

The query is basically inspired on https://wiki.postgresql.org/wiki/Index_Maintenance
