
-- Pg_prewarm Extension
This extension help us to understand how the shared buffer works and in sizing it better.
Excract the maximum performance from this memory component

-- The pg_prewarm module provides a convenient way to load relation data into PostgreSQL buffer cache.  
    Prewarming can be performed in two ways
    1) Manually using the pg_prewarm function
    2) Performed automatically by including pg_prewarm in shared_preload_libraries. 
--- In case of auto prewarm, system will run a background worker which periodically 
    records the contents of shared buffers in a file called autoprewarm.blocks 
    and will be using 2 background workers, reload those same blocks after a restart.

-- Prerequisites:
   * Contrib module needs to be installed in Linux for pg_prewarm extension.

-- Manual Prewarm:
```
CREATE EXTENSION pg_prewarm;

-- Check how many blocks of table available in pg_buffercache.
SELECT count(*)
FROM pg_buffercache
WHERE relfilenode = pg_relation_filenode('labs'::regclass);

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

-- Number of pages used by table.
SELECT oid::regclass AS tbl, relpages
FROM   pg_class
WHERE  relname = 'labs';

-- Call Pg_prewarm extension and load the table.
SELECT * FROM pg_prewarm('labs');

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

-- Restart postgresql.

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
```

-- Auto Prewarm:
```
CREATE EXTENSION pg_prewarm;
ALTER SYSTEM SET shared_preload_libraries = 'pg_prewarm';
-- Check how many blocks of table available in pg_buffercache.
SELECT count(*)
FROM pg_buffercache
WHERE relfilenode = pg_relation_filenode('labs'::regclass);

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

-- Number of pages used by table.
SELECT oid::regclass AS tbl, relpages
FROM   pg_class
WHERE  relname = 'labs';

-- Call Pg_prewarm extension and load the table.
SELECT * FROM pg_prewarm('labs');

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
-- 
