## PG STAT REPLCIATION VIEW is the golden source
```
Pg_stat_replication view (primary server)
    pid:              Process id of walsender process
   usesysid:         OID of user which is used for Streaming replication.
   usename:          Name of user which is used for Streaming replication
   application_name: Application name connected to Primary
   client_addr:      Address of standby/streaming replication
   client_hostname:  Hostname of standby.
   client_port:      TCP port number on which standby communicating with WAL sender
   backend_start:    Start time when SR connected to Primary.
   state:            Current WAL sender state i.e streaming
   sent_location:    Last transaction location sent to standby.
   write_location:   Last transaction written on disk at standby
   flush_location:   Last transaction flush on disk at standby.
   replay_location:  Last transaction flush on disk at standby.
   sync_priority:    Priority of standby server being chosen as synchronous standby
   sync_state:       Sync State of standby (is it async or synchronous).
```
