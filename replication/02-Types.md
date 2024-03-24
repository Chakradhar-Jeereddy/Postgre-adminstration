```
Physical replication
  File-based log shipping
  Streaming replication
  Physical replocation replicate all databases
  Cannot replicate between two different major versions or platforms, minor version difference is ok.

Logical replication
  Replicating data objects and their changes based upon their replication identity(Usually a primary key).
  Support table-level data synchronization
  Consolidating multiple databasases into one single entity. (Analytics)

Standby types
 Warm standby - No read access, can't be used to offload read operations from productions system.
 Hot standby - Can be used for read operations.
```
  
