# Case 2:
## Objective: Test automatic failover and ensure primary is joined back to repmgr after initial failover
### Instance : 
### Primary : 192.168.1.8
### Standby 192.168.1.9

Steps:
1)	Verify the current status of nodes and the roles
```
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
```
2)	Shutdown postgresql on primary server and verify the repmgrd logs
```
syntax: systemctl stop postgresql-12
tail -100f /var/lib/pgsql/repmgr.log
```
3)	Verify the current status of nodes and the roles
```
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
```
4)	Start primary node and check the status of both the nodes in cluster show
```
syntax: systemctl stop postgresql-12
```

## How to add the failed primary as new standby.
### Steps: 
1)	stop the primary database which has recovered from failure.
2)	Set /usr/pgsql-12/bin in the bash profile.
Export PATH=/us/pgsql-12/bin:$PATH
3)	Rejoin the node .
4)	On Standby server ( which is the new primary) perform a checkpoint
```
./rpmgr -f /var/lib/pgsql/repmgr.conf node service --action=stop --checkpoint
``` 
5)	Start postgresql on standby
6)	Now execute node join on primary with dry run to check everything is working fine
```
./repmgr node rejoin -f /var/lib/pgsql/repmgr.conf -d 'host=192.168.1.9 user=repmgr dbname=repmgr --force-rewind
--config-files=postgresql.local.conf,postgresql.conf --verbose --dry-run
7)	Perform the node join operation.
8)	Now check the show cluster.
```
./rpmgr -f /var/lib/pgsql/repmgr.conf
```

 
