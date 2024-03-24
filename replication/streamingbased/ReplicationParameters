- Wal_level: Replica - determines how much information is written to the WAL.
          - Default value is replica, which writes enough data to support WAL archiving and replication, including running
            read-only queries on a standby server.
- Wal_log_hints = on - required for pg_rewind capability when the standby goes out of sync with master.
- Max_wal_senders: integer - Specifies the maximum number of concurrent connections from standby server or streaming base 
                    backup clients (the maximum number of simultaneously running wal sender processes). 
                    Default is 10. Value 0 means replication is disabled.
- Wal_keep_segments: integer - Specifies the minimum number of past log file segments kept in pg_wal directory, in case a 
                               standby server needs to fetch them for streaming replication. If a standby server connected to
                               sending server might remove a WAL segment still needed by the standby, in which case the
                               replication connection will be terminated.
- host_standby = on - Enables read only connection on the node when it is in standby role. This is ignored when the server is
                      running as master. Standby server will begin accepting read only connections once the recovery has 
                      brought the system to a consistent state.
