- PG_STAT_STATEMENTS

- pg_stat_statements module  provides a means for tracking planning and execution statistics of all SQL statements.
  This extension is not available globally but can be enabled for a specific database with CREATE EXTENSION pg_stat_statements.

- Configuration:
  shared_preload_libraries = 'pg_stat_statements'
  pg_stat_statements.max = 10000
  pg_stat_statements.track = all
  track_activity_query_size = 2048
  track_io_timing = on

- PG_STAT_STATEMENTS.MAX: 
  Maximum number of statements tracked by the module.

- PG_STAT_STATEMENTS.TRACK:
  Controls which statement the module tracks. 
        top (track statements issued directly by clients)
        all (track top-level and nested statements), 
        and none (disable statement statistics collection).

- Track_IO_Timing:
  Enables timing of database I/O calls.
  This parameter is off by default, as it will repeatedly query the operating system for the current time,
  which may cause significant overhead on some platforms. 

- Track_Activity_Query_Size:
  parameter sets the number of characters to display when reporting a SQL query. The default value is 1024 bytes.


- PG_STAT_STATEMENT_RESET:
  Discards all statistics gathered so far by pg_stat_statements.
  By default, this function can only be executed by superusers.
