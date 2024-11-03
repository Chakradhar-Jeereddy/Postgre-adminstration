- PG_STAT_STATEMENTS_EXAMPLES
- Long Running Queries in Postgresql:
```
WITH statements AS (
SELECT * FROM pg_stat_statements pss
		JOIN pg_roles pr ON (userid=oid)
WHERE rolname = current_user
)
SELECT calls, 
	mean_exec_time, 
	query
FROM statements
WHERE calls > 0
AND shared_blks_hit > 0
ORDER BY mean_exec_time DESC
LIMIT 10;
```
- Top Queries based on Cpu Usage:
```
SELECT 
pss.userid,
pss.dbid,
pd.datname as db_name,
round((pss.total_exec_time + pss.total_plan_time)::numeric, 2) as total_time, 
pss.calls, 
round((pss.mean_exec_time+pss.mean_plan_time)::numeric, 2) as mean, 
round((100 * (pss.total_exec_time + pss.total_plan_time) /
sum((pss.total_exec_time + pss.total_plan_time)::numeric) OVER ())::numeric, 2) as cpu_portion_pctg,
substr(pss.query, 1, 200) short_query
FROM pg_stat_statements pss, pg_database pd 
WHERE pd.oid=pss.dbid
ORDER BY (pss.total_exec_time + pss.total_plan_time)
DESC LIMIT 30;
```
- Top queries based on memory usage:
```
 select userid::regrole, dbid, query
    from pg_stat_statements
    order by (shared_blks_hit+shared_blks_dirtied) desc
    limit 10;
```
- Top I/O intensive queries:
```
select userid::regrole, dbid, query
    from pg_stat_statements
    order by (blk_read_time+blk_write_time)/calls desc
    limit 10;
```
- Top 10 consumers of temporary space:
```
select userid::regrole, dbid, query 
    from pg_stat_statements
    order by temp_blks_written desc
    limit 10;
```
- Top 10 Based on Total Execution Time:
```
SELECT userid::regrole, dbid, (total_exec_time / 1000 / 60) as total_min,   mean_exec_time as avg_ms,
calls,query 
FROM pg_stat_statements 
ORDER BY total_exec_time 
DESC LIMIT 10;   
```
- Cache hit ratio:
```
SELECT query, calls, total_exec_time, rows, 100.0 * shared_blks_hit /
               nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
          FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 10;
```
- Cache Hit ratio with Shared_Blks_Hits,Shared_blks_read:
```
WITH statements AS (
SELECT * FROM pg_stat_statements pss
		JOIN pg_roles pr ON (userid=oid)
WHERE rolname = current_user
)
SELECT calls, 
	shared_blks_hit,
	shared_blks_read,
	shared_blks_hit/(shared_blks_hit+shared_blks_read)::NUMERIC*100 hit_cache_ratio,
	query
FROM statements
WHERE calls > 0
AND shared_blks_hit > 0
ORDER BY calls DESC, hit_cache_ratio ASC
LIMIT 10;
```
- Detail I/o Information:
```
select
	shared_blks_hit + shared_blks_read + shared_blks_dirtied + shared_blks_written +
local_blks_hit + local_blks_read + local_blks_dirtied + local_blks_written + temp_blks_read +
temp_blks_written as total_buffers,
	(total_exec_time + total_plan_time)::int as total_time,
	calls,
	shared_blks_hit as sbh,
	shared_blks_read as sbr,
	shared_blks_dirtied as sbd, 
	shared_blks_written as sbw,
	local_blks_hit as lbh,
	local_blks_read as lbr,
	local_blks_dirtied as lbd,
	local_blks_written as lbw,
	temp_blks_read as tbr,
	temp_blks_written as tbw,
	query
from
	pg_stat_statements
order by
	total_buffers desc
limit 10;


Reset pg_stat_statements:
SELECT pg_stat_statements_reset();
```


