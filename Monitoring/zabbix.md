Zabbix scripts
```
select count(*) from pg_prepared_xacts;

select count(*) from pg_stat_replicaton;

select row_to_json(T) from (select current_setting('max_connections')) T;
shared_buffer
effactive_cache_size
maintenance_work_mem
checkpoint_completion_target
wal_buffers
default_statistics_target
random_page_cost
work_mem
effective_io_concurrency
max_wal_size
max_worker_processes
shared_preload_library

with T as (select sum(CASE when relkind in ('r','t','m') then pg_stat_get_numscans(oid) seq,
                  sum(CASE when relkind in ('i') then pg_stat_get_numscans(oid) idx from pg_class
                  where relkind in ('r','t','m','i') ) select row_to_json(T) from T;

select current_setting('autovacuum_freeze_max_age')

select row_to_json(T) from (select indexrelname as index_name,idx_blks_read,idx_blks_hit from pg_statio_user_indexes order by idx_blks_read desc limit 10) T;

show server_version;
select checkpoints_timed,checkpoints_req,checkpoint_write_time,checkpoint_sync_time,buffers_checkpoint*8192,buffers_backend*8192,buffers_backend_fsync,buffers_alloc*8192 from pg_stat_bgwriter;

Few more to scripts to monitor.....

```
