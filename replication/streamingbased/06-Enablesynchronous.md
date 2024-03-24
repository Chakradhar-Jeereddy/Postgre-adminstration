## Synchronous Replication
- The transaction is commited in standby before committed in master for data protection.
- Synchronous replication offers the ability to commit a transaction (or write data) to the primary database and the standby/replica simultaneously.
- Transaction are considered successful when all changes made by the transaction have been transferred to one or more synchronous standby servers.
```
Syntax to enable Synchronous 
     ALTER SYSTEM SET synchronous_standby_names TO  '*‘
     Reload PostgreSQL 12 service to apply the new changes.
     Note - alter system command updates the parameter in postgresql.auto.conf file.
            After every restart this file is read and the values of the parameter are applied.
```
- Verify the sync mode is enabled
```
    select * from pg_stat_replication;
    state: sync
```
- Remove sync mode
```
alter system reset all;
restart the cluster
select * from pg_stat_replication;
state: async
```

## Note: Use synchronous replication when standby reliable or when using more then one standby.
   
