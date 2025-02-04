- pg_ctl can be used to start, stop and promote/failover a standby database to primary. (Note: Standby can accept only reads in PG)

- Startup commands
```
pg_ctl -D $PGDATA start  #Unlike oracle, no muliple startup modes in postgresql
pg_ctl -D $PGDATA status
```
- A cluster is started using one data directory and a port on just one host.

# Shutdown modes - default is smart
- Smart - It waits for the existing connections to disconnect, new ones will not be allowed
- Fast  - It will kill existing connections and does checkpoint and rollback to avoid crash recovery during next startup, this mode is widely used in pord
- Immediate - This is simliar to shut abort in oracle, which explains the behaviour.
- we use -m flag to specify the mode of shutdown
  ```
  pg_ctl -D $PGDATA stop -ms
  pg_ctl -D $PGDATA stop -mf
  pg_ctl -D $PGDATA stop -mi
  ```
- Note - never use kill command to stop the cluster.
  
# Ways to find data directory
- if the cluster is running
- ps -ef|grep /post   
- ps -ef|grep postgres
- if the cluster is not running
- locate postgresql.auto.conf   - postgrsql.auto.conf can't be stored in different location other than data directory
  
  

  
  
  
