---
title: "Performance"
date: 2021-11-09T18:19:52+01:00
draft: true
---


Connect to the database as the owner (postgres user in my case). 

Check that you're on the database you want to monitor, then enable the `pg_stat_statements` extension.

```postgres
# Double check current database
select current_database();

# Enable the extension
CREATE EXTENSION pg_stat_statements;
```


----

Interesting read: https://www.cybertec-postgresql.com/en/a-quick-pg_stat_statements-troubleshooting-hack/
