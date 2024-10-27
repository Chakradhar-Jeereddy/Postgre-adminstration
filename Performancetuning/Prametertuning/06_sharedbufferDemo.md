```
Work Mem :  4MB Default
select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
SET work_mem = '64MB';
select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
SET work_mem = '1GB';
select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
RESET work_mem;
```
```
Use this for demo: 
explain analyze select * from pgbench_history order  by aid;
set work_mem='10MB';
SET log_temp_files TO '4MB';
SET trace_sort TO 'on';  (To include resource information).
```	
```
A well-known formula suggests :
25% of the total system memory/ max_connections.


select relname,last_vacuum, last_autovacuum, last_analyze, vacuum_count, autovacuum_count,
  last_autoanalyze from pg_stat_user_tables where schemaname = 'micro' order by relname ASC;


ALTER ROLE usernameA SET work_mem TO '1GB'

It's tough to get the right value for work_mem perfect, but often a sane default can be something like 64 MB, 
```
