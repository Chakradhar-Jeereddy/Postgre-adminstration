-- Checkpoint ensures all dirty buffers created up to a certain point are sent to disk so the wall
   upto that point can be recycled.
-- It is a point in walfile sequence at will all data files are updated to reflect the information in the log.

-- When checkpoint is triggered?
  * if checkpoint_timeout value is reached.
  * Max_wal_size limit is reached default value is 1GB
  * when user manually issues a checkpoint
  * during startup and shutdown of cluster
  * When pg_start_backup is called.
  * When a database is created

```
postgres=# show checkpoint_timeout
postgres-# ;
-[ RECORD 1 ]------+-----
checkpoint_timeout | 5min

postgres=# show checkpoint_completion_target;
-[ RECORD 1 ]----------------+----
checkpoint_completion_target | 0.9

postgres=# show max_wal_size;
-[ RECORD 1 ]+----
max_wal_size | 1GB
```
- Checkpoint_timeout: 5 minutes(default)
- After every 5 mins checkpoint is triggered and dirty pages are written to disk.
- Incease this will increase the amount of time required for crash recovery.
If set to 5 mins
   * Pros 1) Faster after crash recovery. Since less work will need to be redone.
           2) IF no wal files generated since previous checkpoint, the new checkpoint is skipped event after the timeout is passed
     Cros: Frequent checkpointing/writing pages disk can cause a significant I/O load causing huge performance impact.

  Idle value for checkpoint is 20 minutes.
  3 checkpoints per hour.

  * checkpoint_completion_target = 0.9 => if checkpoint is every 10mins it says to complete the task in 9minutes.
    remaining 1 min it will wait for next timeout to occur.

* max_wal_size - each walfile is 16mb in size,if file reaches 16mb another file gets created. After this 1gb
                 is reached it will trigger a checkpoint(it is called requested checkpoint, we need to avoid as
                 it is not expected checkpoint and cause performance issues. After checkpoint, the walfiles are
                 recycled.
  * Ensure majority of checkpoints should be timed based checkpoints rather than requested ones.
  * Ensure the wal sizing is right. In 20 minuites how many wal files are generated what is the size of wal size
    and increase the max_wal_size parameter to reduce requested checkpoint.
    * Increasing max_wal_size, the recovery time after crash also increases.

    -- How to monitor requested and timed checkpoints, use pg_stat_bgwriter;
    
  
  
