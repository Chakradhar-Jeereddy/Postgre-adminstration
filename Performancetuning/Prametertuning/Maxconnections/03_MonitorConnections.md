### Useful Pg_stat_activity Queries for Monitoring Connections:

-- Number of connected user:
```
SELECT usename AS username FROM pg_stat_activity where usename!=''  GROUP BY usename;
```
--  Which user and how many concurrent connections:
```
SELECT usename AS username, count(*) AS concurrent_statements
FROM pg_stat_activity
WHERE state = 'active'
GROUP BY usename;
```
-- If you need to figure out where the connections are going, you can break down
the connections by database.
```
SELECT datname, numbackends FROM pg_stat_database;
```

-- investigate connections to a specific database, query pg_stat_activity.
```
SELECT * FROM pg_stat_activity WHERE datname='postgres';
```
-- All active connections but not the current query:
```
SELECT
    age(clock_timestamp(), query_start),
    usename,
    datname,
    query
FROM pg_stat_activity
WHERE
    state != 'idle'
AND query NOT ILIKE '%pg_stat_activity%'
ORDER BY age desc;
```

-- All processes that are not idle but do have a wait event:
```
SELECT
    usename,
    datname,
    query,
    wait_event_type,
    wait_event
FROM pg_stat_activity
WHERE
    state != 'idle'
AND wait_event != '';
```
-- Query backend_type equal to client_backend.
```
SELECT * FROM pg_stat_activity WHERE backend_type = 'client backend';
```
-- Query to find start, state,state_change,pid and duration
```
SELECT pid, now() - query_start AS duration, query_start, state_change, state, query FROM pg_stat_activity WHERE backend_type = 'client backend';
```
-- Session which are running for more 10 seconds and are not idle
```
select
now()-query_start as runtime,
pid as process_id,
datname as db_name,client_addr,client_hostname,
query
from pg_stat_activity
where state!='idle'
and now() - query_start > '10 seconds':: interval
order by 1 desc;
```
-- Kill connections based on time frame:
```
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'Database_Name'
	AND pid <> pg_backend_pid()
	AND state in ('idle', 'idle in transaction', 'idle in transaction (aborted)', 'disabled') 
	AND state_change < current_timestamp - INTERVAL '15' MINUTE;
```

-- Postgres kill all idle in transaction
```
SELECT pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE datname='db'
  AND state = 'idle in transaction';
```

-- To kill all active connections to a PostgreSQL database, execute the query below:
```
SELECT
 pg_terminate_backend(pid)
FROM
 pg_stat_activity
WHERE
 datname ='postgres'
AND
 leader_pid 
IS NULL;
```
-- query to kill all connections except for yours:
```
SELECT
 pg_terminate_backend(pid)
FROM
 pg_stat_activity
WHERE
 datname = 
'postgres'
AND
 pid != pg_backend_pid()
AND
 leader_pid 
IS NULL;
```

-- To terminate all connections to all databases in a Postgres server(except yours), 
run the following query:
```
SELECT
 pg_terminate_backend(pid)
FROM
 pg_stat_activity
WHERE
 pid != pg_backend_pid()
AND
 datname 
IS NOT NULL
AND
 leader_pid 
IS NULL;
```
-- specific session:
```
select pid, query from pg_stat_activity where datname = current_database();
select pg_terminate_backend(123);
```
