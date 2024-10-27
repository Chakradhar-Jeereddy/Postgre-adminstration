-- Generate tables in postgresql database using pgbench
```
../init/bin/pgbench -i -s 100
postgres=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
 public | test1            | table | postgres
```
-- Check table size
```
\dt+
                                         List of relations
 Schema |       Name       | Type  |  Owner   | Persistence | Access method |  Size | Description
--------+------------------+-------+----------+-------------+---------------+---------+-------------
 public | pgbench_accounts | table | postgres | permanent   | heap          | 1281 MB|
 public | pgbench_branches | table | postgres | permanent   | heap          | 40 kB|
 public | pgbench_history  | table | postgres | permanent   | heap          | 0 bytes|
 public | pgbench_tellers  | table | postgres | permanent   | heap          | 80 kB|
 public | test1            | table | postgres | permanent   | heap          | 0 bytes|
```

-- Restart postgresql cluster
```
stop
start
status
```

-- Read data from pgbench_accounts to see if it gets from buffer of disk
```
 explain (analyze,buffers) select * from pgbench_accounts;
                                                           QUERY PLAN
 Seq Scan on pgbench_accounts  (cost=0.00..263935.00 rows=10000000 width=97) (actual
time=0.012..1083.376 rows=10000000 loops=1)
   Buffers: shared read=163935   (All read from disk)
 Planning:
   Buffers: shared hit=44 read=11
 Planning Time: 0.360 ms
 Execution Time: 1478.041 ms
(6 rows)



explain (analyze,buffers) select * from pgbench_accounts;
                                                           QUERY PLAN


 Seq Scan on pgbench_accounts  (cost=0.00..263935.00 rows=10000000 width=97) (actual
time=0.031..1296.835 rows=10000000 loops=1)
   Buffers: shared hit=32 read=163903   => Only 32 buffers of 8k got allocated
 Planning Time: 0.042 ms
 Execution Time: 1763.781 ms
(4 rows)

 explain (analyze,buffers) select * from pgbench_accounts;
                                                           QUERY PLAN

 Seq Scan on pgbench_accounts  (cost=0.00..263935.00 rows=10000000 width=97) (actual
time=0.032..1448.150 rows=10000000 loops=1)
   Buffers: shared hit=64 read=163871    Only 64 buffers of 8k got allocated
 Planning Time: 0.041 ms
 Execution Time: 1976.656 ms

explain (analyze,buffers) select * from pgbench_accounts;
                                                           QUERY PLAN

-------------------------------------------------------------------------------------
--------------------------------------------
 Seq Scan on pgbench_accounts  (cost=0.00..263935.00 rows=10000000 width=97) (actual
time=0.024..1335.968 rows=10000000 loops=1)
   Buffers: shared hit=96 read=163839    Only 96 buffers of 8k got allocated
 Planning Time: 0.034 ms
 Execution Time: 1824.100 ms

explain (analyze,buffers) select * from pgbench_accounts;
                                                           QUERY PLAN

-------------------------------------------------------------------------------------
--------------------------------------------
 Seq Scan on pgbench_accounts  (cost=0.00..263935.00 rows=10000000 width=97) (actual
time=0.028..1437.912 rows=10000000 loops=1)
   Buffers: shared hit=128 read=163807      128 buffers of 8k got allocated
 Planning Time: 0.041 ms
 Execution Time: 1971.269 ms
(4 rows)

Note - Every time the query is executed additonal 32 buffers gets allocated
```

```

-- Now 
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
