```
Single-master/unidirectional replication
 Changes to master are replicatated to one or more replica databases.
 Tables in replica are not permitted to accept any changes(except from master)

Multi-master/bidirectional replication
Changes to tables in more than one designated master database are replicated to their counterpart tables in every other master database.
Conflict resolutions are applied to avoid problems like duplicate primary key.
```
