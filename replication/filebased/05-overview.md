```
Overview-
WAL files are shipped from the master to the standby servers to keep data in sync.
Master can directly copy the logs to standby server storage or can share storage with the standby servers.
The primary server operates in continious archiving mode, while each standby server operates in continious recovery mode, reading the WAL files from the primary.
FIle-based log shipping is asynchronous.
One wal log file can contain upto 16MB of data.
The WAL file is shipped only after it reaches that threshold.
This will cause a delay in replication and also increase chances of losing data if the master crashes and logs are not archived.
```
