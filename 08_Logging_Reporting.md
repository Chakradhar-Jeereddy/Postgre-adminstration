## Error Reporting and Logging

- Where to Log:
•	Logging_collector: This parameter enables the logging collector,
                     which is a background process that captures log messages sent to stderr
                     and redirects them into log files. It requires a restart and it starts a logger process.

•	log_directory: When logging_collector is enabled, this parameter determines the directory in which log files will be created. 
                 It can be specified as an absolute path, or relative to the cluster data directory.  


•	log_filename: When logging_collector is enabled, this parameter sets the file names of the created log files. 
alter system set log_filename='postgresql-%a.log';
%H – Hours of the day.
%a is the day of the week
%w is the week of the month
%d day of the month 
Examples:
1)	%H – Hours - log_filename = 'postgresql-%H.log', set log_rotation_age = 60
2)	For backup period of one week, use %a-%w: log_filename = 'postgresql-%a.log', and set log_rotation_age = 1440.
3)	one log file per day named server_log.Mon, server_log.Tue, etc., and automatically overwrite last week's log with this week's log,
4)	set log_filename to server_log.%a, log_truncate_on_rotation to on.
5)	For backup period of one month, use %d: log_filename = 'postgresql-%d.log', and set log_rotation_age = 1440.

log_truncate_on_rotation to on, the existing data of the file is not appended to the new file.
. When to Log:
 log_min_duration_statement : Causes the duration of each completed statement to be logged if the statement ran for at least the specified amount of time. 

. What to Log:
•	log_statement
This is the degree of logging you're using. The level of detail you want in your logs is commonly referred to as log levels. 
There are several options for log_statement, including ddl (which solely records database structure changes), 
mod (which records changes to existing data), and all (logs everything).

•	log_checkpoints
  Checkpoints in PostgreSQL are periodic activities that store data about your system, as we described in the configuration settings. 
  Excessive use of log checkpoints can result in performance degradation. If you suspect this is the case, 
  enable log checkpoints to get detailed information about the checkpoints, including how often they run and what might be triggering them.
  It logs following information - 
   - Checkpoint starttime - Indicates when the checkpoint process starts, along with the reason(eg time-based, requested or
     segment bases).
   - Buffer written: The number of dirty buffers (modified data pages) written to disk.
   - WAL Files
     Added: New WAL (Write-Ahead logs) files created during the checkpoint.
     Removed: Old wal files removed
     Recycled: Existing wal files reused.
     Durations:
       Write: Time taken to write dirty pages to disk.
       Sync: Time spent synchronizing changes to disk.
       Total: Total time taken for the checkpoint process.

Sync files:
 Count: Number of files synchronized during checkpoint.
 Longest: Longest time taken to sync files
 Average: Average time taken to sync files

 Distance:
  Number of wal bytes between the current checkpoint and previous checkpoint.
  Estimate: Predicate distance for the next checkpoint.

   
•	log_connection
  You might also be interested in knowing about links. Something could be wrong if you just have one application connected to your database, 
  but you notice a lot of concurrent connections. Too many connections flooding your database can cause requests to fail to reach the database, 
  causing problems for your application's end users.

•	log_autovacuum_min_duration
  Causes each action executed by auto vacuum to be logged if it ran for at least the specified amount of time.  
  The default is 10min.  For example, if you set this to 250ms then all automatic vacuums and analyzes that run 250ms or longer will be logged. 
  Message will be logged if an auto vacuum action is skipped due to a conflicting lock or a concurrently dropped relation.

•	Log_disconnections
  Causes session terminations to be logged. The log output provides information like log_connections, plus the duration of the session. The default is off.

•	log_temp_files 
  Controls logging of temporary file names and sizes. Temporary files can be created for sorts, hashes, and temporary query results.  
  If enabled by this setting, a log entry is emitted for each temporary file, with the file size specified in bytes, when it is deleted. 
  A value of zero logs all temporary file information, while positive values log only files whose size is greater than or equal to the specified amount of data.

•	log_line_prefix
    This is a printf-style string that is output at the beginning of each log line.  
    Example :'time=%t, pid=%p %q db=%d, usr=%u, client=%h , app=%a, line=%l'
