# 统计信息

**查看索引的使用情况**

```sql
SELECT indexrelname,cast(idx_tup_read AS numeric) / idx_scan
   AS avg_tuples,idx_scan,idx_tup_read FROM pg_stat_user_indexes where idx_scan > 0;
```

***

**查看索引的大小**

```sql
SELECT
         schemaname,
         relname,
         indexrelname,
         idx_scan,
         pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size
       FROM
         pg_stat_user_indexes i
         JOIN pg_index USING (indexrelid)
       WHERE
         indisunique IS false
       ORDER BY idx_scan,relname;
```

***

**查看链接情况**

```sql
SELECT * FROM pg_stat_activity;
```


*查看活动状态*

```sql
SELECT query, wait_event_type, wait_event
      FROM pg_stat_activity
```

***

**查看锁的情况**

```sql

SELECT
         locktype,
         virtualtransaction,
         transactionid,
         nspname,
         relname,
         mode,
         granted,
         cast(date_trunc('second',query_start) AS timestamp) AS query_start,
         substr(query,1,25) AS query
       FROM
         pg_locks
           LEFT OUTER JOIN pg_class ON (pg_locks.relation = pg_class.oid)
           LEFT OUTER JOIN pg_namespace ON (pg_namespace.oid =
   pg_class.relnamespace),
         pg_stat_activity
       WHERE
         NOT pg_locks.pid=pg_backend_pid() AND
         pg_locks.pid=pg_stat_activity.
             pid
;```

