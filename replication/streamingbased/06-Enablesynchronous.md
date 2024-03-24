## Synchronous Replication

- Synchronous replication offers the ability to commit a transaction (or write data) to the primary database and the standby/replica simultaneously.
- Transaction are considered successful when all changes made by the transaction have been transferred to one or more synchronous standby servers.
```
Syntax to enable Synchronous 
     ALTER SYSTEM SET synchronous_standby_names TO  '*‘
     Reload PostgreSQL 12 service to apply the new changes.
```

