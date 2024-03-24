## Streaming Failover

- Failover is the ability of a system to continue functioning even if some failure occurs.
- Functioning of the system are assumed by secondary components if the primary components fail.
- PostgreSQL in itself does not provide an automatic failover solution.
- We can manually failover postgresql from master to server using below mentioned 3 methods:
```
           ./pg_ctl promote -D /var/lib/pgsql/12/data
           Create a trigger file with the file name and path specified by the promote_trigger_file.
           SELECT pg_promote();  --> it returns 't', which indicates promoted to master.
           SELECT pg_is_in_recovery();  -> it should return 'f'
 ```
- Log entries after execution of promot command
```
received promot request
terminating wal receiver process
archive recovery complete
database system is ready to accept connections
```

## Re-establish replication
- Build old master as new standby.
- create replication slot in new primary
