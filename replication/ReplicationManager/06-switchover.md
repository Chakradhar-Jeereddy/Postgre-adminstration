
![switchover!](switchover.jpg)

- Go to standby and run the command below
```
./repmgr standby switchover -f /var/lib/pgsql/repmgr.conf --siblings-follow --dry-run
Note - siblings-follow used if you have another standby that needs to follow new primary after switchover
./repmgr standby switchover -f /var/lib/pgsql/repmgr.conf --siblings-follow (actual run)
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
```
- After completion of maintenance on old primary, do the switchover on that server
```
./repmgr standby switchover -f /var/lib/pgsql/repmgr.conf --siblings-follow --dry-run
./repmgr standby switchover -f /var/lib/pgsql/repmgr.conf --siblings-follow (actual run)
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show --compact
./repmgr -f /var/lib/pgsql/repmgr.conf node service pause (it will not do automatic failover)
./repmgrd -f /var/lib/pgsql/repmgr.conf (it start repmgrd deamon which montiors the node status in custer status)
```

Switchover
```
Dry run

-bash-4.2$ /usr/pgsql-12/bin/repmgr standby switchover -f /u01/12.1/repmgr.conf  --dry-run
NOTICE: checking switchover on node "primary" (ID: 1) in --dry-run mode
INFO: SSH connection to host "replica" succeeded
INFO: able to execute "repmgr" on remote host "replica"
INFO: 1 walsenders required, 10 available
INFO: demotion candidate is able to make replication connection to promotion candidate
INFO: 0 pending archive files
INFO: replication lag on this standby is 0 seconds
NOTICE: attempting to pause repmgrd on 2 nodes
INFO: would pause repmgrd on node "primary" (ID: 1)
INFO: would pause repmgrd on node "replica" (ID: 2)
NOTICE: local node "primary" (ID: 1) would be promoted to primary; current primary "replica" (ID: 2) would be demoted to standby
INFO: following shutdown command would be run on node "replica":
  "/usr/pgsql-12/bin/pg_ctl  -D '/u01/12.1/data' -W -m fast stop"
INFO: parameter "shutdown_check_timeout" is set to 60 seconds
INFO: prerequisites for executing STANDBY SWITCHOVER are met

Actual run

/usr/pgsql-12/bin/repmgr standby switchover -f /u01/12.1/repmgr.conf
NOTICE: executing switchover on node "replica" (ID: 2)
NOTICE: attempting to pause repmgrd on 2 nodes
NOTICE: local node "replica" (ID: 2) will be promoted to primary; current primary "primary" (ID: 1) will be demoted to standby
NOTICE: stopping current primary node "primary" (ID: 1)
NOTICE: issuing CHECKPOINT on node "primary" (ID: 1)
DETAIL: executing server command "/usr/pgsql-12/bin/pg_ctl  -D '/u01/12.1/data' -W -m fast stop"
INFO: checking for primary shutdown; 1 of 60 attempts ("shutdown_check_timeout")
INFO: checking for primary shutdown; 2 of 60 attempts ("shutdown_check_timeout")
NOTICE: current primary has been cleanly shut down at location 2/F0000028
NOTICE: promoting standby to primary
DETAIL: promoting server "replica" (ID: 2) using pg_promote()
NOTICE: waiting up to 60 seconds (parameter "promote_check_timeout") for promotion to complete
NOTICE: STANDBY PROMOTE successful
DETAIL: server "replica" (ID: 2) was successfully promoted to primary
NOTICE: node "replica" (ID: 2) promoted to primary, node "primary" (ID: 1) demoted to standby
NOTICE: switchover was successful
DETAIL: node "replica" is now primary and node "primary" is attached as standby
NOTICE: STANDBY SWITCHOVER has completed successfully

```


Standby clone:
```
Dry run -
-bash-4.2$  /usr/pgsql-12/bin/repmgr standby clone -h primary -p 5434 -d repmgr -U repmgr -f /u01/12.1/repmgr.conf --dry-run
NOTICE: destination directory "/u01/12.1/data" provided
INFO: connecting to source node
DETAIL: connection string is: host=primary port=5434 user=repmgr dbname=repmgr
DETAIL: current installation size is 1528 MB
INFO: "repmgr" extension is installed in database "repmgr"
INFO: replication slot usage not requested;  no replication slot will be set up for this standby
INFO: parameter "max_wal_senders" set to 10
NOTICE: checking for available walsenders on the source node (2 required)
INFO: sufficient walsenders available on the source node
DETAIL: 2 required, 10 available
NOTICE: checking replication connections can be made to the source server (2 required)
INFO: required number of replication connections could be made to the source server
DETAIL: 2 replication connections required
NOTICE: standby will attach to upstream node 1
HINT: consider using the -c/--fast-checkpoint option
INFO: would execute:
  /usr/pgsql-12/bin/pg_basebackup -l "repmgr base backup"  -D /u01/12.1/data -h primary -p 5434 -U repmgr -X stream
INFO: all prerequisites for "standby clone" are met

Actual run -
-bash-4.2$  /usr/pgsql-12/bin/repmgr standby clone -h primary -p 5434 -d repmgr -U repmgr -f /u01/12.1/repmgr.conf
NOTICE: destination directory "/u01/12.1/data" provided
INFO: connecting to source node
DETAIL: connection string is: host=primary port=5434 user=repmgr dbname=repmgr
DETAIL: current installation size is 1528 MB
INFO: replication slot usage not requested;  no replication slot will be set up for this standby
NOTICE: checking for available walsenders on the source node (2 required)
NOTICE: checking replication connections can be made to the source server (2 required)
INFO: checking and correcting permissions on existing directory "/u01/12.1/data"
NOTICE: starting backup (using pg_basebackup)...
HINT: this may take some time; consider using the -c/--fast-checkpoint option
INFO: executing:
  /usr/pgsql-12/bin/pg_basebackup -l "repmgr base backup"  -D /u01/12.1/data -h primary -p 5434 -U repmgr -X stream
NOTICE: standby clone (using pg_basebackup) complete
NOTICE: you can now start your PostgreSQL server
HINT: for example: pg_ctl -D /u01/12.1/data start
HINT: after starting the server, you need to re-register this standby with "repmgr standby register --force" to update the existing node record
```

Node rejoin dry run (Issue checkpoint on new primary)
```
/usr/pgsql-12/bin/repmgr -f /u01/12.1/repmgr.conf node service --action=stop --checkpoint
/usr/pgsql-12/bin/repmgr -f /u01/12.1/repmgr.conf node service --action=start --checkpoint

-bash-4.2$  /usr/pgsql-12/bin/repmgr node rejoin -d 'host=primary port=5434 dbname=repmgr user=repmgr' -f /u01/12.1/repmgr.conf --force-rewind --config-files=postgresql.conf --dry-run
NOTICE: rejoin target is node "primary" (ID: 1)
INFO: replication connection to the rejoin target node was successful
INFO: local and rejoin target system identifiers match
DETAIL: system identifier is 7460724183491783790
NOTICE: pg_rewind execution required for this node to attach to rejoin target node 1
DETAIL: rejoin target server's timeline 5 forked off current database system timeline 4 before current recovery point 2/EC000028
INFO: prerequisites for using pg_rewind are met
INFO: file "postgresql.conf" would be copied to "/tmp/repmgr-config-archive-replica/postgresql.conf"
INFO: pg_rewind would now be executed
DETAIL: pg_rewind command is:
  /usr/pgsql-12/bin/pg_rewind -D '/u01/12.1/data' --source-server='host=primary user=repmgr dbname=repmgr port=5434 connect_timeout=10'
INFO: prerequisites for executing NODE REJOIN are met

Actual run

-bash-4.2$  /usr/pgsql-12/bin/repmgr node rejoin -d 'host=primary port=5434 dbname=repmgr user=repmgr' -f /u01/12.1/repmgr.conf --force-rewind --config-files=postgresql.conf
NOTICE: rejoin target is node "primary" (ID: 1)
NOTICE: pg_rewind execution required for this node to attach to rejoin target node 1
DETAIL: rejoin target server's timeline 5 forked off current database system timeline 4 before current recovery point 2/EC000028
NOTICE: executing pg_rewind
DETAIL: pg_rewind command is "/usr/pgsql-12/bin/pg_rewind -D '/u01/12.1/data' --source-server='host=primary user=repmgr dbname=repmgr port=5434 connect_timeout=10'"
ERROR: pg_rewind execution failed
DETAIL: pg_rewind: servers diverged at WAL location 2/EC000000 on timeline 4
pg_rewind: error: could not open file "/u01/12.1/data/pg_wal/0000000400000002000000EB": No such file or directory
pg_rewind: fatal: could not find previous WAL record at 2/EB001E40
```


