# Replication Slots
- It is used to retain WAL files in master when standby goes offline or out of sync.
- Master server uses replication slots to keep track of how much the standby lags and retain WAL it needs until the standby 
   reconnects again.
- Replication slot came in with PostgreSQL9.4, before that wal_keep_segment parameter used to govern how many wal files need to
  be maintained.
- The slots needs to be created manually and the default value is 10.
- Replication slots are of two types
     - Physical replication slots
     - Logical replication slots
