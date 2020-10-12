# 命令

**删除用户和表空间**

```sql
drop user DATACENTER cascade;
drop tablespace DATACENTER including contents and datafiles;
```

***

**查看表空间使用情况**

```sql
select total.TABLESPACE_NAME,
       round(total.MB, 2) as TOTAL_MB,
       round(total.MB - free.MB, 2) as USED_MB,
       ROUND((1 - free.MB / total.MB) * 100, 2) || '%' as USED_PCT,
       ROUND(free.MB, 2) as FREE_mb
  from (select a.TABLESPACE_NAME, sum(a.BYTES) / 1024 / 1024 as MB
          from sys.dba_data_files a
         group by a.TABLESPACE_NAME) total,
       (select b.TABLESPACE_NAME,
               count(1) as extends,
               sum(b.BYTES) / 1024 / 1024 as MB,
               sum(b.BLOCKS) as blocks
          from sys.dba_free_space b
         group by b.TABLESPACE_NAME) free
 where total.TABLESPACE_NAME = free.TABLESPACE_NAME
```

***

**删除session**

```sql
select 'alter system kill session ''' || sid || ',' || serial# || ''' immediate;' from v$session 
where username= 'HDC_ODS';
```

***

**查看oracle日志路径**

```sql
SELECT * FROM v$logfile ORDER BY group#; 
```

