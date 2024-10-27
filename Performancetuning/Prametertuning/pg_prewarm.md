
* Pg_prewarm Extension
* This extension help us to understand how the shared buffer works and in sizing it better.
* Excract the maximum performance from this memory component

* The pg_prewarm module provides a convenient way to load relation data into PostgreSQL buffer cache.  
    Prewarming can be performed in two ways
    1) Manually using the pg_prewarm function
    2) Performed automatically by including pg_prewarm in shared_preload_libraries.
    3) Although table is pinned, if there is space pressure and table not requently used, it gets flushed.
* In case of auto prewarm, system will run a background worker which periodically 
    records the contents of shared buffers in a file called autoprewarm.blocks 
    and will be using 2 background workers, reload those same blocks after a restart.

* Prerequisites:
   * Contrib module needs to be installed in Linux for pg_prewarm extension.

-- Manual Prewarm:
```
postgres=# create extension pg_prewarm;
CREATE EXTENSION
postgres=# \dx
                      List of installed extensions
      Name      | Version |   Schema   |           Description
----------------+---------+------------+---------------------------------
 pg_buffercache | 1.4     | public     | examine the shared buffer cache
 pg_prewarm     | 1.2     | public     | prewarm relation data
 plpgsql        | 1.0     | pg_catalog | PL/pgSQL procedural language
(3 rows)


-- Check how many blocks of table available in pg_buffercache.
-- No make sure table not in buffer restart cluster
SELECT count(*)
FROM pg_buffercache
WHERE relfilenode = pg_relation_filenode('pgbench_accounts'::regclass);
 count
-------
     0

-- Check the tables in buffer cache.
SELECT c.relname,
  count(*) blocks,
  round( 100.0 * 8192 * count(*) / pg_table_size(c.oid) ) "% of rel",
  round( 100.0 * 8192 * count(*) FILTER (WHERE b.usagecount > 3) / pg_table_size(c.oid) ) "% hot"
FROM pg_buffercache b
  JOIN pg_class c ON pg_relation_filenode(c.oid) = b.relfilenode
WHERE  b.reldatabase IN (
         0, (SELECT oid FROM pg_database WHERE datname = current_database())
       )
AND    b.usagecount is not null
GROUP BY c.relname, c.oid
ORDER BY 2 DESC
LIMIT 10;

-- Number of pages used by pgbench_accounts.
SELECT oid::regclass AS tbl, relpages
FROM   pg_class
WHERE  relname = 'pgbench_accounts';
       tbl        | relpages
------------------+----------
 pgbench_accounts |   163935

-- Call Pg_prewarm extension and load the table.
-- Any table which is used regularly is good candidate for pg_prewarm
SELECT * FROM pg_prewarm('pgbench_accounts');
 pg_prewarm
------------
     164033

-- Check the table is in memory:
SELECT c.relname,
  count(*) blocks,
  round( 100.0 * 8192 * count(*) / pg_table_size(c.oid) ) "% of rel",
  round( 100.0 * 8192 * count(*) FILTER (WHERE b.usagecount > 3) / pg_table_size(c.oid) ) "% hot"
FROM pg_buffercache b
  JOIN pg_class c ON pg_relation_filenode(c.oid) = b.relfilenode
WHERE  b.reldatabase IN (
         0, (SELECT oid FROM pg_database WHERE datname = current_database())
       )
AND    b.usagecount is not null
GROUP BY c.relname, c.oid
ORDER BY 2 DESC
LIMIT 10;

             relname             | blocks | % of rel | % hot
---------------------------------+--------+----------+-------
 pgbench_accounts                |   1640 |      100 |   100
 pg_attribute                    |     32 |       49 |    46
 pg_class                        |     14 |       78 |    78
 pg_proc                         |     13 |       12 |     2
 pg_operator                     |     11 |       61 |    39
 pg_attribute_relid_attnum_index |      9 |       75 |    67
 pg_proc_oid_index               |      9 |       82 |    27
 pg_proc_proname_args_nsp_index  |      6 |       19 |     3
 pg_amproc                       |      5 |       56 |    11
 pg_amop                         |      5 |       45 |    27

explain (analyze,buffers) select * from pgbench_accounts;
                                                       QUERY PLAN

 Seq Scan on pgbench_accounts  (cost=0.00..2640.00 rows=100000 width=97) (actual tim
e=0.004..9.120 rows=100000 loops=1)
   Buffers: shared hit=1640    => No disk reads
 Planning:
   Buffers: shared hit=60
 Planning Time: 0.231 ms
 Execution Time: 12.117 ms
(6 rows)

-- Restart postgresql.
stop/start
-- Check the table is in memory: (The table wont be their in memory)
SELECT c.relname,
  count(*) blocks,
  round( 100.0 * 8192 * count(*) / pg_table_size(c.oid) ) "% of rel",
  round( 100.0 * 8192 * count(*) FILTER (WHERE b.usagecount > 3) / pg_table_size(c.oid) ) "% hot"
FROM pg_buffercache b
  JOIN pg_class c ON pg_relation_filenode(c.oid) = b.relfilenode
WHERE  b.reldatabase IN (
         0, (SELECT oid FROM pg_database WHERE datname = current_database())
       )
AND    b.usagecount is not null
GROUP BY c.relname, c.oid
ORDER BY 2 DESC
LIMIT 10;
             relname             | blocks | % of rel | % hot
---------------------------------+--------+----------+-------
 pg_attribute                    |     31 |       48 |    45
 pg_proc                         |     12 |       11 |     2
 pg_operator                     |     11 |       61 |    39
 pg_class                        |     10 |       56 |    56
 pg_proc_oid_index               |      9 |       82 |    27
 pg_attribute_relid_attnum_index |      8 |       67 |    50
 pg_proc_proname_args_nsp_index  |      6 |       19 |     3
 pg_amproc                       |      5 |       56 |    11
 pg_amop                         |      5 |       45 |    27
 pg_operator_oprname_l_r_n_index |      5 |       83 |    33

explain (analyze,buffers) select * from pgbench_accounts;
                                                       QUERY PLAN

 Seq Scan on pgbench_accounts  (cost=0.00..2640.00 rows=100000 width=97) (actual tim
e=0.009..15.984 rows=100000 loops=1)
   Buffers: shared read=1640      => Read from disk
 Planning:
   Buffers: shared hit=25 read=3
 Planning Time: 0.116 ms
 Execution Time: 19.086 ms

Issue with manual approach is the table gets removed from the shared buffers after restart.
```

-- Auto Prewarm:

```
CREATE EXTENSION pg_prewarm;

ps -ef|post
postgres  11491      1  0 02:00 ?        00:00:00 /u01/16.2/init/bin/postgres -D /u01/16.2/data
postgres  11492  11491  0 02:00 ?        00:00:00 postgres: logger
postgres  11493  11491  0 02:00 ?        00:00:00 postgres: checkpointer
postgres  11494  11491  0 02:00 ?        00:00:00 postgres: background writer
postgres  11496  11491  0 02:00 ?        00:00:00 postgres: walwriter
postgres  11497  11491  0 02:00 ?        00:00:00 postgres: autovacuum launcher
postgres  11498  11491  0 02:00 ?        00:00:00 postgres: logical replication launcher

ALTER SYSTEM SET shared_preload_libraries = 'pg_prewarm';
-- The prewarming can increase the startup time of cluster

ps -ef|grep post
postgres  11741      1  0 02:16 ?        00:00:00 /u01/16.2/init/bin/postgres -D /u01/16.2/data
postgres  11742  11741  0 02:16 ?        00:00:00 postgres: logger
postgres  11743  11741  0 02:16 ?        00:00:00 postgres: checkpointer
postgres  11744  11741  0 02:16 ?        00:00:00 postgres: background writer
postgres  11746  11741  0 02:16 ?        00:00:00 postgres: walwriter
postgres  11747  11741  0 02:16 ?        00:00:00 postgres: autovacuum launcher
postgres  11748  11741  0 02:16 ?        00:00:00 postgres: autoprewarm leader   ========> this one
postgres  11749  11741  0 02:16 ?        00:00:00 postgres: logical replication launcher


-- Check how many blocks of table available in pg_buffercache.
SELECT count(*)
FROM pg_buffercache
WHERE relfilenode = pg_relation_filenode('pgbench_accounts'::regclass);
 count
-------
     0

-- Check the table is present in buffer cache.
SELECT c.relname,
  count(*) blocks,
  round( 100.0 * 8192 * count(*) / pg_table_size(c.oid) ) "% of rel",
  round( 100.0 * 8192 * count(*) FILTER (WHERE b.usagecount > 3) / pg_table_size(c.oid) ) "% hot"
FROM pg_buffercache b
  JOIN pg_class c ON pg_relation_filenode(c.oid) = b.relfilenode
WHERE  b.reldatabase IN (
         0, (SELECT oid FROM pg_database WHERE datname = current_database())
       )
AND    b.usagecount is not null
GROUP BY c.relname, c.oid
ORDER BY 2 DESC
LIMIT 10;
             relname             | blocks | % of rel | % hot
---------------------------------+--------+----------+-------
 pg_attribute                    |     31 |       48 |    45
 pg_proc                         |     12 |       11 |     2
 pg_operator                     |     11 |       61 |    39
 pg_class                        |     10 |       56 |    56
 pg_proc_oid_index               |      9 |       82 |    27
 pg_attribute_relid_attnum_index |      8 |       67 |    50
 pg_proc_proname_args_nsp_index  |      6 |       19 |     3
 pg_amproc                       |      5 |       56 |    11
 pg_amop                         |      5 |       45 |    27
 pg_operator_oprname_l_r_n_index |      5 |       83 |    33

-- Number of pages used by table.
SELECT oid::regclass AS tbl, relpages
FROM   pg_class
WHERE  relname = 'pgbench_accounts';
       tbl        | relpages
------------------+----------
 pgbench_accounts |     1640

-- Call Pg_prewarm extension and load the table.
SELECT * FROM pg_prewarm('pgbench_accounts');


-- Check the table is in memory:
SELECT c.relname,
  count(*) blocks,
  round( 100.0 * 8192 * count(*) / pg_table_size(c.oid) ) "% of rel",
  round( 100.0 * 8192 * count(*) FILTER (WHERE b.usagecount > 3) / pg_table_size(c.oid) ) "% hot"
FROM pg_buffercache b
  JOIN pg_class c ON pg_relation_filenode(c.oid) = b.relfilenode
WHERE  b.reldatabase IN (
         0, (SELECT oid FROM pg_database WHERE datname = current_database())
       )
AND    b.usagecount is not null
GROUP BY c.relname, c.oid
ORDER BY 2 DESC
LIMIT 10;
             relname             | blocks | % of rel | % hot
---------------------------------+--------+----------+-------
 pgbench_accounts                |   1640 |      100 |     0
 pg_attribute                    |     32 |       49 |    45
 pg_class                        |     14 |       78 |    78
 pg_proc                         |     13 |       12 |     2
 pg_operator                     |     11 |       61 |    39
 pg_attribute_relid_attnum_index |      9 |       75 |    50
 pg_proc_oid_index               |      9 |       82 |    27
 pg_proc_proname_args_nsp_index  |      6 |       19 |     3
 pg_amproc                       |      5 |       56 |    11
 pg_amop                         |      5 |       45 |    27

-- Restart postgresql.

-- Check the table is in memory: (The table should be in their memory)
SELECT c.relname,
  count(*) blocks,
  round( 100.0 * 8192 * count(*) / pg_table_size(c.oid) ) "% of rel",
  round( 100.0 * 8192 * count(*) FILTER (WHERE b.usagecount > 3) / pg_table_size(c.oid) ) "% hot"
FROM pg_buffercache b
  JOIN pg_class c ON pg_relation_filenode(c.oid) = b.relfilenode
WHERE  b.reldatabase IN (
         0, (SELECT oid FROM pg_database WHERE datname = current_database())
       )
AND    b.usagecount is not null
GROUP BY c.relname, c.oid
ORDER BY 2 DESC
LIMIT 10;

             relname             | blocks | % of rel | % hot
---------------------------------+--------+----------+-------
 pgbench_accounts                |   1640 |      100 |     0  => still there
 pg_attribute                    |     32 |       49 |    45
 pg_class                        |     14 |       78 |    56
 pg_proc                         |     13 |       12 |     2
 pg_operator                     |     11 |       61 |    39
 pg_attribute_relid_attnum_index |      9 |       75 |    50
 pg_proc_oid_index               |      9 |       82 |    27
 pg_proc_proname_args_nsp_index  |      6 |       19 |     3
 pg_amproc                       |      5 |       56 |    11
 pg_amop                         |      5 |       45 |    27

-- If you want to know how much percentage of the buffer the table is using:
SELECT
  c.relname,
  pg_size_pretty(count(*) * 8192) as buffered,
  round(100.0 * count(*) / 
    (SELECT setting FROM pg_settings
      WHERE name='shared_buffers')::integer,1)
    AS buffers_percent,
  round(100.0 * count(*) * 8192 / 
    pg_table_size(c.oid),1)
    AS percent_of_relation
FROM pg_class c
  INNER JOIN pg_buffercache b
    ON b.relfilenode = c.relfilenode
  INNER JOIN pg_database d
    ON (b.reldatabase = d.oid AND d.datname = current_database())
GROUP BY c.oid,c.relname
ORDER BY 3 DESC LIMIT 10;

     relname      |  buffered  | buffers_percent | percent_of_relation
------------------+------------+-----------------+---------------------
 pgbench_accounts | 13 MB      |            10.0 |                99.8
 pg_operator      | 88 kB      |             0.1 |                61.1
 pg_amop          | 48 kB      |             0.0 |                54.5
 pg_cast          | 16 kB      |             0.0 |                33.3
 pg_constraint    | 8192 bytes |             0.0 |                12.5
 pg_index         | 32 kB      |             0.0 |                50.0
 pg_opclass       | 16 kB      |             0.0 |                28.6
 pg_namespace     | 8192 bytes |             0.0 |                16.7
 pg_amproc        | 40 kB      |             0.0 |                55.6
 pg_am            | 8192 bytes |             0.0 |                20.0

-- 
