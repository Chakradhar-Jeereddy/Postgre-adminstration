-- PG_BUFFERCACHE to inspect shared_buffers
```
The pg_buffercache module provides a means for examining what's happening in the shared buffer cache in real time.
 use is restricted to superusers and roles with privileges of the pg_monitor role. Access may be granted to others using GRANT.
```
-- Test Cases:
```
How to install pg_buffercache: (Linux user please install contrib module)
Check if extention is installed using the command \dx
postgres=# \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description
---------+---------+------------+------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(1 row)

postgres=# create extension pg_bffercache;
ERROR:  extension "pg_bffercache" is not available
DETAIL:  Could not open extension control file "/u01/16.2/init/share/postgresql/extension/pg_bffercache.control": No such file or directory.
HINT:  The extension must first be installed on the system where PostgreSQL is running.

```
-- Install contrib as root
```
cd /u01/postgresql-16.2/contrib
make
make install

postgres=# Create extension pg_buffercache;
CREATE EXTENSION

postgres=# \dx
                      List of installed extensions
      Name      | Version |   Schema   |           Description
----------------+---------+------------+---------------------------------
 pg_buffercache | 1.4     | public     | examine the shared buffer cache
 plpgsql        | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)

-- Extendtion installed successfully to examine workload
```
-- Lets run some workload using pg_bench
```
pgbench -c 100 -j 2 -U postgres postgres
pgbench (16.2)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 100
query mode: simple
number of clients: 100
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10
number of transactions actually processed: 1000/1000
number of failed transactions: 0 (0.000%)
latency average = 164.089 ms
initial connection time = 717.274 ms
tps = 609.424259 (without initial connection time)
```
-- Examine the shared buffers
```
-- Check database buffercache for all cache blocks in each database:
SELECT CASE WHEN c.reldatabase IS NULL THEN ''
            WHEN c.reldatabase = 0 THEN ''
            ELSE d.datname
       END AS database,
       count(*) AS cached_blocks
FROM  pg_buffercache AS c
      LEFT JOIN pg_database AS d
           ON c.reldatabase = d.oid
GROUP BY d.datname, c.reldatabase
ORDER BY d.datname, c.reldatabase;

 database  | cached_blocks
-----------+---------------
 postgres  |          2846
 template1 |            71
 template2 |            14
           |         13453  (This are empty blocks(important), if it is not there then shared buffer is not sized properly)
When we have lots of empty blocks we don't have to worry about it.

```
-- Check how many blocks are empty/dirty/clean:
```
SELECT buffer_status, sum(count) AS count
  FROM (SELECT CASE isdirty
                 WHEN true THEN 'dirty'
                 WHEN false THEN 'clean'
                 ELSE 'empty'
               END AS buffer_status,
               count(*) AS count
          FROM pg_buffercache
          GROUP BY buffer_status
        UNION ALL
          SELECT * FROM (VALUES ('dirty', 0), ('clean', 0), ('empty', 0)) AS tab2 (buffer_status,count)) tab1
  GROUP BY buffer_status;
buffer_status | count
---------------+-------
 clean         |  2105  (pinned and released)
 dirty         |   831  (to be writen to disk after checkpoint and auto no eviction)
 empty         | 13448

Issue Checkpoint: (Run the above query again and check how many pages are dirty)
 Checkpoint;
 buffer_status | count
---------------+-------
 clean         |  2936
 dirty         |     0  (It become zero, becuase those were written to disk)
 empty         | 13448

```
-- In the current database how many table are cache and how many buffer used.
```
postgres=# select current_database();
 current_database
------------------
 postgres

SELECT n.nspname, c.relname, count(*) AS buffers
             FROM pg_buffercache b JOIN pg_class c
             ON b.relfilenode = pg_relation_filenode(c.oid) AND
                b.reldatabase IN (0, (SELECT oid FROM pg_database
                                      WHERE datname = current_database()))
             JOIN pg_namespace n ON n.oid = c.relnamespace
             GROUP BY n.nspname, c.relname
             ORDER BY 3 DESC
             LIMIT 10;
  nspname   |        relname        | buffers
------------+-----------------------+---------
 public     | pgbench_accounts      |    1260
 public     | pgbench_accounts_pkey |    1076
 public     | pgbench_history       |     102
 public     | pgbench_tellers       |      91
 public     | pgbench_branches      |      38
 pg_catalog | pg_attribute          |      32
 pg_catalog | pg_proc               |      21
 pg_catalog | pg_class              |      17
 pg_catalog | pg_operator           |      14
 pg_catalog | pg_proc_oid_index     |      10

 ```
-- Inspect Individual table in buffer cache.
```
SELECT * FROM pg_buffercache WHERE relfilenode = pg_relation_filenode('pgbench_history');
-[ RECORD 1 ]----+------
bufferid         | 801
relfilenode      | 16475
reltablespace    | 1663
reldatabase      | 5
relforknumber    | 0
relblocknumber   | 0
isdirty          | f
usagecount       | 5
pinning_backends | 0
```
--- Inspect buffer cache for tables and indexes which are cache:
```
SELECT c.relname, c.relkind, count(*)
       FROM   pg_database AS a, pg_buffercache AS b, pg_class AS c
       WHERE  c.relfilenode = b.relfilenode
              AND b.reldatabase = a.oid  
              AND c.oid >= 16384
              AND a.datname = 'postgres'
       GROUP BY 1, 2
       ORDER BY 3 DESC, 1;
        relname        | relkind | count
-----------------------+---------+-------
 pgbench_accounts      | r       |  1260
 pgbench_accounts_pkey | i       |  1076
 pgbench_history       | r       |   102
 pgbench_tellers       | r       |    91
 pgbench_branches      | r       |    38
 pgbench_tellers_pkey  | i       |     7
 pgbench_branches_pkey | i       |     2

```
-- Inspect buffer cache to know how much portion of table/index is buffered, in percentage and in terms of relation:
```
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
ORDER BY 3 DESC
LIMIT 10;
        relname        |  buffered  | buffers_percent | percent_of_relation
-----------------------+------------+-----------------+---------------------
 pgbench_accounts      | 10080 kB   |             7.7 |                 0.8
 pgbench_accounts_pkey | 8608 kB    |             6.6 |                 3.9
 pgbench_history       | 816 kB     |             0.6 |               100.0
 pgbench_tellers       | 728 kB     |             0.6 |               100.0
 pgbench_branches      | 304 kB     |             0.2 |               100.0
 pg_statistic          | 80 kB      |             0.1 |                29.4
 pg_operator           | 112 kB     |             0.1 |                77.8
 pg_cast               | 16 kB      |             0.0 |                33.3
 pg_amproc             | 40 kB      |             0.0 |                55.6
 pg_aggregate          | 8192 bytes |             0.0 |                14.3
```

-- Find all blocks and their usage count:
```
SELECT
 c.relname, count(*) AS buffers,usagecount
FROM pg_class c
 INNER JOIN pg_buffercache b
 ON b.relfilenode = c.relfilenode
 INNER JOIN pg_database d
 ON (b.reldatabase = d.oid AND d.datname = current_database())
GROUP BY c.relname,usagecount
ORDER BY c.relname,usagecount;
                    relname                     | buffers | usagecount
------------------------------------------------+---------+------------
 pg_aggregate                                   |       1 |          4
 pg_aggregate_fnoid_index                       |       1 |          2
 pg_aggregate_fnoid_index                       |       1 |          4
 pg_am                                          |       1 |          5
 pg_amop                                        |       2 |          2
 pg_amop                                        |       5 |          5
 pg_amop_fam_strat_index                        |       3 |          5
```

-- Distribution of blocks based on usage_count:
```
SELECT usagecount, count(*)
FROM pg_buffercache
GROUP BY usagecount
ORDER BY usagecount;
 usagecount | count
------------+-------
          1 |   227
          2 |  1037
          3 |   990
          4 |    35
          5 |   664
            | 13431
```
-- How much percentage of table/index is cache and how much % is hot:
```
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

        relname        | blocks | % of rel | % hot
-----------------------+--------+----------+-------
 pgbench_accounts      |   1260 |        1 |     0
 pgbench_accounts_pkey |   1076 |        4 |     0
 pgbench_history       |    102 |      100 |    99
 pgbench_tellers       |     91 |      100 |    53
 pgbench_branches      |     38 |      100 |    89
 pg_attribute          |     33 |       52 |    49
 pg_proc               |     26 |       24 |    10
 pg_class              |     18 |      100 |    78
 pg_operator           |     14 |       78 |    72
 pg_statistic          |     10 |       29 |    15

```
