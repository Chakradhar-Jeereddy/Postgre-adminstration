pg_stat_bgwriter is a view which provides metrics about how Postgresql flushes dirty buffers to the disk?

There are 3 ways how the dirty buffers are flushed.
Checkpoint - buffer_checkpoint(column)
background_writer --buffer_clean
backends ------> buffer_backends

 A comparison of buffers_checkpoint, buffers_clean, and buffers_backend is insightful for understanding 
 the distribution of writes during checkpoints, 
 by the background writer, and in backend sessions, respectively.


 PG_STAT_BGWRITER

pg_stat_bgwriter is a view which provides metrics about how PostgreSQL flushes dirty buffers to the disk.
There are 3 ways how the dirty buffers are flushed.
 * Checkpoint --> buffer_checkpoint (column)
 * Background_Writer ---> buffer_clean
 * Backends ----> buffer_backends

 A comparison of buffers_checkpoint, buffers_clean, and buffers_backend is insightful for understanding the distribution of writes during checkpoints, by the background writer, and in backend sessions, respectively.

Checkpoint -> Timed or Requested. 
Timed checpoints basically happens due to when the checkpoint_timeout is achieved and this are basically desirable.
Requested checkpoints are inherently unpredictable and mostly happens if max_wal_size is breached. 

Tip:
Aim to keep majority of checkpoints as timed checkpoints and reduce requested checkpoint.
This can be achieved by setting a max_wal_size to a higher value  or if the checkpoint_timeout value is too large. We can go ahead and reduce the timing of checkpoint_timeout.

checkpoint_write_time : Total time spent in the checkpoint processing portion when writing files to disk, in milliseconds.

checkpoint_sync_time: Total time spent in the checkpoint processing portion when synchronizing files to disk, in milliseconds.

Checkpoint --> buffer_checkpoint : Number of buffers written by checkpoints.  

Tip:
For Better Performance it is advisable to have a majority of buffers written to the disk during checkpoints. so a higher number for buffer_checkpoint is preferable over backends or by the background writer.

Background_Writer ---> buffer_clean : Number of buffers written by the background writer process. A high buffers_clean value implies effective workload reduction during checkpoints by the background writer.

bgwriter_delay : 200ms 
bgwriter_lru_maxpages : 100
bgwriter_lru_multiplier: 2

The maxwritten_clean metric shows the frequency at which the background writer halts due to reaching its maxpages limit. If the value is high, try to increase bgwriter_lru_maxpages for flushing more writes per round.

Backends ----> buffer_backends : Number of buffers written directly by the backend. High buffers_backend can also indicate extensive bulk insert or update operations. Ensure to keep buffers_backend as low as possible , as high values suggest that PostgreSQL sessions are taking on tasks typically handled by the background writer. This might indicate a need for more shared_buffers, or a more aggressive background writer configuration, by adjusting bgwriter_lru_maxpages, bgwriter_lru_multiplier, and reducing bgwriter_delay. 

stats_reset : Time at which these statistics were last reset.
Command to Reset Statistics
Select pg_stat_reset_shared('bgwriter');




PG_STAT_BGWRITER columns
Name	Type	Description
checkpoints_timed	bigint	Number of scheduled checkpoints that have been performed.
checkpoints_req	bigint	Number of checkpoints that have been performed actively.
checkpoint_write_time	double precision	Total time spent in the checkpoint processing portion when writing files to disk, in milliseconds.
checkpoint_sync_time	double precision	Total time spent in the checkpoint processing portion when synchronizing files to disk, in milliseconds.
buffers_checkpoint	bigint	Number of buffers written by checkpoints.
buffers_clean	bigint	Number of buffers written by the background writer process.
maxwritten_clean	bigint	Number of times that cleanup scanning stops because the background writer process writes too many buffers.
buffers_backend	bigint	Number of buffers written directly by the backend.
buffers_backend_fsync	bigint	Number of times that the backend calls fsync (usually, even if the backend executes these write actions, the background writer process processes them again).
buffers_alloc	bigint	Number of buffers allocated.
stats_reset	timestamp with time zone	Time at which these statistics were last reset.

