
Auto Vacuum  Common Issues and Resolutions:

How Autovacuum cost is calculated:
Autovacuum reads pages looking for dead tuples, and if none are found, autovacuum discards the page. 
When autovacuum finds dead tuples, it removes them. The cost is based on:
•	vacuum_cost_page_hit: Cost of reading a page that is already in shared buffers and doesn't need a disk read. The default value is set to 1.
•	vacuum_cost_page_miss: Cost of fetching a page that isn't in shared buffers. The default value is set to 2.
•	vacuum_cost_page_dirty: Cost of writing to a page when dead tuples are found in it. The default value is set to 20.


Common Issues and Solutions:
Issue: Autovacuum running slow:   
1)	Tables are getting vacuum slow and Vacuum process constantly appear in pg_stat_activity.
SELECT query FROM pg_stat_activity WHERE backend_type = 'autovacuum worker';
           
  Resolution :
1)	maintenance_work_mem \Autovacuum_work_mem: Increase to allow each autovacuum worker process to store more dead tuples while scanning a table.
2)	autovacuum_vacuum_cost_delay:Decrease to reduce cost limiting sleep time and make vacuuming faster.
3)	autovacuum_vacuum_cost_limit: Increase the cost to be accumulated before vacuum will sleep, thereby reducing sleep frequency and making vacuum go faster.
(Good for Large number of Databases in Cluster).
4)	autovacuum_max_workers	Increase to allow more parallel workers to be triggered by autovacuum.

Issue: Autovacuum not happening enough.
1)	   SELECT relname, last_vacuum, last_autovacuum FROM pg_stat_user_tables;

Resolution: 
1)	autovacuum_vacuum_scale_factor	Lower the value to trigger vacuuming more frequently, useful for larger tables with more updates / deletes.
2)	autovacuum_vacuum_insert_scale_factor	Lower the values to trigger vacuuming more frequently for large, insert-heavy tables.

Issue: Autovacuum is consuming too much system resource.
1)	Spike in system resources  memory/ Disk i-o
2)	Slow other query performance.

Resolution:
1)	Increase autovacuum_vacuum_cost_delay and reduce autovacuum_vacuum_cost_limit if set higher than the default of 200.
2)	Reduce the number of autovacuum_max_workers if it's set higher than the default of 

Issue: Vacuum does not clean up dead rows efficiently.
1)	Tables are not getting vacuum properly and dead rows still show up.

Resolution:
1)	Check for long running transaction which block vacuum process.
2)	Termination long running transaction helps in freeing up dead tuples for deletion. 
3)	Query to check long running transaction.
   SELECT pid, age(backend_xid) AS age_in_xids,
    now () - xact_start AS xact_age,
    now () - query_start AS query_age,
    state,
    query
    FROM pg_stat_activity
    WHERE state != 'idle'
    ORDER BY 2 DESC
    LIMIT 10;          



Monitor Autovacuum:
1)Find Auto vacuum is turned on or not:
SELECT name, setting FROM pg_settings WHERE name='autovacuum';

2) Find how many dead rows in a table:
SELECT relname, n_dead_tup FROM pg_stat_user_tables;

3) Find Track-Count is turned on or not (Enables collection of statistics on database activity)
SELECT name, setting FROM pg_settings WHERE name='track_counts';

4) Check Autovacuum is enabled on table level:
SELECT reloptions FROM pg_class WHERE relname='Tablename';

5) Check parameter settings related with autovacuum.:
SELECT * from pg_settings where category like 'Autovacuum';

6) Find  when was a table last vacuum/Auto vacuumed:
SELECT relname, last_vacuum, last_autovacuum FROM pg_stat_user_tables;

7) To check progress of a running vacuum: 
select * from pg_stat_progress_vacuum;

8) Dead Tuples percentage /Last Autovacuum.
select schemaname,relname,n_dead_tup,n_live_tup,round(n_dead_tup::float/n_live_tup::float*100) dead_pct,autovacuum_count,last_vacuum,last_autovacuum,last_autoanalyze,last_analyze from pg_stat_all_tables where n_live_tup >0;

9) Tables currently qualify for vacuum:
SELECT *
      ,n_dead_tup > av_threshold AS av_needed
      ,CASE
        WHEN reltuples > 0
          THEN round(100.0 * n_dead_tup / (reltuples))
        ELSE 0
        END AS pct_dead
    FROM (
      SELECT N.nspname
        ,C.relname
        ,pg_stat_get_tuples_inserted(C.oid) AS n_tup_ins
        ,pg_stat_get_tuples_updated(C.oid) AS n_tup_upd
        ,pg_stat_get_tuples_deleted(C.oid) AS n_tup_del
        ,pg_stat_get_live_tuples(C.oid) AS n_live_tup
        ,pg_stat_get_dead_tuples(C.oid) AS n_dead_tup
        ,C.reltuples AS reltuples
        ,round(current_setting('autovacuum_vacuum_threshold')::INTEGER + current_setting('autovacuum_vacuum_scale_factor')::NUMERIC * C.reltuples) AS av_threshold
        ,date_trunc('minute', greatest(pg_stat_get_last_vacuum_time(C.oid), pg_stat_get_last_autovacuum_time(C.oid))) AS last_vacuum
        ,date_trunc('minute', greatest(pg_stat_get_last_analyze_time(C.oid), pg_stat_get_last_autoanalyze_time(C.oid))) AS last_analyze
      FROM pg_class C
      LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
      WHERE C.relkind IN (
          'r'
          ,'t'
          )
        AND N.nspname NOT IN (
          'pg_catalog'
          ,'information_schema'
          )
        AND N.nspname !~ '^pg_toast'
      ) AS av
    ORDER BY av_needed DESC ,n_dead_tup DESC;
