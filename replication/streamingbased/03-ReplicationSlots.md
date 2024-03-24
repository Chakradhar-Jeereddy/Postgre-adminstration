# Replication Slots
- It is used to retain WAL files in master when standby goes offline or out of sync.
- Master server uses replication slots to keep track of how much the standby lags and retain WAL it needs until the standby 
   reconnects again.
- Replication slot came in with PostgreSQL9.4, before that wal_keep_segment parameter used to govern how
   many wal files need to be maintained.
- The slots needs to be created manually and the maximum slots allowed value is 10.
- Replication slots are of two types
     - Physical replication slots
     - Logical replication slots
- 
- How to create a physical replication slot.
```
On master and enable "physical_replication_slot: 'standby" parameter in postgresql file
     Syntax : select pg_create_physical_replication_slot(‘Standby’);
```
 - How to Monitor a replication slot.
```
     Syntax : select * from pg_replication_slots;
```
- How to delete a replication slot. Make sure orphan slots are removed, if not they hog disk space
```
    Orphan replication slot: Active without standby, to find
    Syntax: select slot_name,active from pg_replication_slots;
            slot_name: standby
            active: f
    Syntax : select pg_drop_replication_slot(‘standby’);
```
- How to create logical replication slot.
```
    Syntax: select pg_create_logical_replication_slot(‘Standby’);
```
