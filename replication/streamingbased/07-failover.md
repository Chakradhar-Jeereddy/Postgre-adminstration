## Streaming Failover

- Failover is the ability of a system to continue functioning even if some failure occurs.
- Functioning of the system are assumed by secondary components if the primary components fail.
- PostgreSQL in itself does not provide an automatic failover solution.
- We can manually failover postgresql from master to server using below mentioned methods:
```
           ./pg_ctl promote -D /var/lib/pgsql/12/data
           Create a trigger file with the file name and path specified by the promote_trigger_file.
           SELECT pg_promote(); 
 ```