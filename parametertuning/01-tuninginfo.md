# Introduction to Server Parameters
- Parameter tuning is a process.
- Server configuration parametersaffect the behavior of the database system.
- 'Out of Box' settings are not suitable for all environments. 
- Not all system are designed the same.
- User interaction with the parameters can be segregated as:
          - Via Configuration File
          - Via SQL
          - Via the Shell

# Memory Parameters	 : Shared Buffers
- Shared Buffers is the amount of ram that can be allocated to shared buffers.
- Ideally contains pages being modified or read
- Shared Buffers uses LRU algorithm to flush the pages from this area.
- Pg_buffercache extension shows what is inside shared_buffers.
- Pg_stat_statements shows the block hit and read for each sql.
- pg_statio_user_tables and pg_statio_user_indexes views to see what is in the cache.


# Work Mem 
- Work Mem is used for Complex Sorting or hash tables.
- In-memory sorts are much faster than sorts spilling to disk.
- Default Value is 4MB.
- Memory allocated for each sort operations(ORDER BY,DISTINCT) and merge joins
- Setting this parameter globally can cause very high memory usage as this parameter is used by per user sort operation.

# Maintenance_Work_mem
- Work Mem * Total Sort Operations for all sort operations
- Maintenance_work_mem is a memory setting used for maintenance tasks. 
- Default value is  64MB.
- Setting a large value helps in tasks like :
            - VACUUM
            - RESTORE
            - CREATE INDEX
            - ADD FOREIGN KEY
            - ALTER TABLE.

# Wal_Buffers
- The amount of shared memory used for WAL data that has not yet been written to disk.
- PostgreSQL writes its WAL (write ahead log) record into the buffers and then these buffers are flushed to disk. 
- The contents of the WAL buffers are written out to disk at every transaction commit,
- Default Size is 16MB.
- Higher value is ideal for concurrent connections.

# Effective_Cache_Size
- effective_cache_size provides an estimate of the memory available for disk caching.
- It is just an estimate, no exact actual memory is allocated.
- It instruct the optimizer the amount of cache available in the kernel.
- Lower value will discourage the query planner to use indexes, even if they are helpful.
- Default value is 4GB.
- lower value prefers sequence scans over index scans.

# Checkpoint_timeout:
- Maximum time between automatic WAL checkpoints, in seconds
- Default value is 5 Minutes.
- Increasing this parameter can increase the amount of time needed for crash recovery.
- Frequent checkpointing results in continuous writes to disk.
- More volume of data written to wal logs when checkpoint interval are less.
- Shutdown may take more time when this value is increased.




