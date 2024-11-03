- Query to Find Bloated Tables
```
select 
  schemaname, 
  relname, 
  n_tup_ins, 
  n_tup_upd, 
  n_tup_del, 
  n_live_tup, 
  n_dead_tup, 
  DATE_TRUNC('minute', last_vacuum) last_vacuum, 
  DATE_TRUNC('minute', last_autovacuum) last_autovacuum
from 
  pg_stat_all_tables 
where 
  schemaname = 'public'
order by 
  n_dead_tup desc;
```

- Get indexes of tables
```
select
    t.relname as table_name,
    i.relname as index_name,
    string_agg(a.attname, ',') as column_name
from
    pg_class t,
    pg_class i,
    pg_index ix,
    pg_attribute a
where
    t.oid = ix.indrelid
    and i.oid = ix.indexrelid
    and a.attrelid = t.oid
    and a.attnum = ANY(ix.indkey)
    and t.relkind = 'r'
    and t.relname not like 'pg_%'
group by  
    t.relname,
    i.relname
order by
    t.relname,
    i.relname;
```

-----------------------
-- Right this second --
-----------------------

- Show running queries
```
SELECT pid, age(query_start, clock_timestamp()), usename, query
FROM pg_stat_activity WHERE query != '<IDLE>'
AND query NOT ILIKE '%pg_stat_activity%' ORDER BY query_start desc;
```
- Queries which are running for more than 2 minutes
```
SELECT now() - query_start as "runtime", usename, datname,state, query
 FROM pg_stat_activity
 WHERE now() - query_start > '2 minutes'::interval ORDER BY runtime DESC;
```
- Queries which are running for more than 9 seconds
```
SELECT now() - query_start as "runtime", usename, datname, state, query FROM pg_stat_activity 
WHERE now() - query_start > '9 seconds'::interval ORDER BY runtime DESC;
```

- Kill running query
```
SELECT pg_cancel_backend(procpid);
```
- Kill idle query
```
SELECT pg_terminate_backend(procpid);
```
- Vacuum Command
VACUUM (VERBOSE, ANALYZE);

--------------------
-- Data Integrity --
--------------------

-  Cache Hit Ratio
```
select sum(blks_hit)*100/sum(blks_hit+blks_read) as hit_ratio from pg_stat_database;
-- (perfectly )hit_ration should be > 90%
```

- Table Sizes
```
select relname, pg_size_pretty(pg_total_relation_size(relname::regclass)) as full_size, 
pg_size_pretty(pg_relation_size(relname::regclass)) as 
table_size, pg_size_pretty(pg_total_relation_size(relname::regclass) - pg_relation_size(relname::regclass)) 
as index_size from pg_stat_user_tables order by pg_total_relation_size(relname::regclass) desc limit 10;
```

- Another Table Sizes Query
```
SELECT nspname || '.' || relname AS "relation", pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size"
FROM pg_class C LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace) WHERE nspname NOT
IN ('pg_catalog', 'information_schema') AND C.relkind <> 'i' AND nspname !~ '^pg_toast'
ORDER BY pg_total_relation_size(C.oid) DESC;
```
-- Database Sizes
```
select datname, pg_size_pretty(pg_database_size(datname)) from pg_database order by pg_database_size(datname);
```
- Unused Indexes
```
select * from pg_stat_all_indexes where idx_scan = 0;
-- idx_scan should not be = 0
```
- Write Activity(index usage)
```
select s.relname, pg_size_pretty(pg_relation_size(relid)), coalesce(n_tup_ins,0) + 2 * coalesce(n_tup_upd,0) - 
coalesce(n_tup_hot_upd,0) + coalesce(n_tup_del,0) AS total_writes, (coalesce(n_tup_hot_upd,0)::float * 100 / 
(case when n_tup_upd > 0 then n_tup_upd else 1 end)::float)::numeric(10,2) AS hot_rate, 
(select v[1] FROM regexp_matches(reloptions::text,E'fillfactor=(d+)') as r(v) limit 1) AS fillfactor 
from pg_stat_all_tables s join pg_class c ON c.oid=relid order by total_writes desc limit 50;

-- hot_rate should be close to 100
```
- Does table needs an Index
```
SELECT relname, seq_scan-idx_scan AS too_much_seq, CASE WHEN seq_scan-idx_scan>0 THEN 'Missing Index?' 
ELSE 'OK' END, pg_relation_size(relname::regclass) AS rel_size, seq_scan, idx_scan FROM pg_stat_all_tables 
WHERE schemaname='public' AND pg_relation_size(relname::regclass)>80000 ORDER BY too_much_seq DESC;
```
- Index % usage
```
SELECT relname, 100 * idx_scan / (seq_scan + idx_scan) percent_of_times_index_used,
n_live_tup rows_in_table FROM pg_stat_user_tables ORDER BY n_live_tup DESC;
```
- How many indexes are in cache
```
SELECT sum(idx_blks_read) as idx_read, sum(idx_blks_hit) as idx_hit, (sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit) as ratio FROM pg_statio_user_indexes;
```
- Dirty Pages
```
select buffers_clean, maxwritten_clean, buffers_backend_fsync from pg_stat_bgwriter;
-- maxwritten_clean and buffers_backend_fsyn better be = 0
```
- Sequential Scans
```
select relname, pg_size_pretty(pg_relation_size(relname::regclass)) as size, seq_scan, seq_tup_read, seq_scan / seq_tup_read as seq_tup_avg from pg_stat_user_tables where seq_tup_read > 0 order by 3,4 desc limit 5;
```

- Checkpoints
```
select 'bad' as checkpoints from pg_stat_bgwriter where checkpoints_req > checkpoints_timed;
```
--------------
-- Activity --
--------------
```
select * from pg_stat_activity where state in ('idle in transaction', 'idle in transaction (aborted)');
```
- Waiting Clients
```
select * from pg_stat_activity where waiting;
```
- Waiting Connections for a lock
```
SELECT count(distinct pid) FROM pg_locks WHERE granted = false;
```

- Connections
```
select client_addr, usename, datname, count(*) from pg_stat_activity group by 1,2,3 order by 4 desc;
```
- User Connections Ratio
```
select count(*)*100/(select current_setting('max_connections')::int) from pg_stat_activity;
```
- Average Statement Exec Time
```
select (sum(total_time) / sum(calls))::numeric(6,3) from pg_stat_statements;
```
- Most writing (to shared_buffers) queries
```
select query, shared_blks_dirtied from pg_stat_statements where shared_blks_dirtied > 0 order by 2 desc;
```
- Block Read Time
```
select * from pg_stat_statements where blk_read_time <> 0 order by blk_read_time desc;
```
---------------
-- Vacuuming --
---------------

- Last Vacuum and Analyze time
```
select relname,last_vacuum, last_autovacuum, last_analyze, last_autoanalyze from pg_stat_user_tables;
```
- Total number of dead tuples need to be vacuumed per table
```
select n_dead_tup, schemaname, relname from pg_stat_all_tables;
```
- Total number of dead tuples need to be vacuumed in DB
```
select sum(n_dead_tup) from pg_stat_all_tables;
```
