```
Asynchronous mode of replication
 Changes replicated to replica later in time after those are commited in master.
 Replica server can remain out-of-sync for a certain duration, which is called replication lag.

Synchronous mode of replication
 Changed commited on master only after those are replicated to replicas.
 Replica server must be available all time and acknowledge for the changes to complete on the master.
```
