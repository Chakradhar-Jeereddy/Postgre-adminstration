#Overview
- The standby server connects to the master server to pull/receive wall records as they are generated.
- Streaming of wal records need not wait for the wal file to be filled.
- Allows standby server to stay more up-to-date than is possible with file-based log shipping.
- By default, streaming replication is asynchronous even though it also supports synchronous replication.

```
Master----> wal sender -------------------wal records------------------wal receiver-----> Slave server
```
