# Case 1:
### Objective:
- Install postgresql and repmgr. 
- Register primary server with repmgr and clone standby server.
- Verify streaming replication.
- Register standby server with server and verify both primary and secondary.

- Instance  Details : 
               - Primary Server : 192.168.1.8
               - Standby Server : 192.168.1.9
### Steps :
- Yum list modules repmgr12*
- Yum -Y install repmgr12.x86_64 ( version 5.2)
- Modify the following parameters in postgresql.conf file
- Max_wal_senders=10
- Max_replication_slots=10
- Wal_level= replica 
- Hot_standby = on
- Archive_command =’bin/true’
- Listen_addresses=’*’
- Shared_preload_libraries= ‘repmgr’
- Wal_log_hints= on
- Start Postgres on primary.
- Create user repmgr as superuser 
- Create database repmgr with owner repmgr
- Configure pg_hba.conf and include repmgr 
- Create a repmgr file
```
node_id=1
node_name=primary
conninfo='host=192.168.1.8 user=repmgr dbname=repmgr connect_timeout=2'
data_directory='/var/lib/pgsql/12/data/'
failover=automatic
promote_command='/usr/pgsql-12/bin/repmgr standby promote -f /var/lib/pgsql/repmgr.conf --log-to-file'
follow_command='/usr/pgsql-12/bin/repmgr standby follow -f /var/lib/pgsql/repmgr.conf --log-to-file --upstream-node-id=%n'
pg_bindir='/usr/pgsql-12/bin'
log_file='/var/lib/pgsql/repmgr.log'
```

- Restart postgresql .
- Register primary server to repmgr
cd /usr/pgsql-12/bin
./repmgr -f /var/lib/pgsql/repmgr.conf primary register
- Verify whether primary server is register with repmgr
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
- Create repmgr file in standby server
```
node_id=2
node_name=standby
conninfo='host=192.168.1.9 user=repmgr dbname=repmgr connect_timeout=2'
data_directory='/var/lib/pgsql/12/data/'
failover=automatic
promote_command='/usr/pgsql-12/bin/repmgr standby promote -f /var/lib/pgsql/repmgr.conf --log-to-file'
follow_command='/usr/pgsql-12/bin/repmgr standby follow -f /var/lib/pgsql/repmgr.conf --log-to-file --upstream-node-id=%n'
pg_bindir='/usr/pgsql-12/bin'
log_file='/var/lib/pgsql/repmgr.log'
```
- Drop all the content of standby data directory.
- Run a dry run to test the configuration.
    /usr/pgsql-12/bin/repmgr -h 192.168.1.8 -U repmgr -f /var/lib/pgsql/repmgr.conf standby clone –dry-run
- Start the clone of standby server 
     /usr/pgsql-12/bin/repmgr -h 192.168.1.8 -U repmgr -f /var/lib/pgsql/repmgr.conf standby clone
- Start postgresql on standby server.
- Register standby server with repmgr.
    cd /usr/pgsql-12/bin
    ./repmgr -f /var/lib/pgsql/repmgr.conf standby register
- Verify whether standby server is register with repmgr and standby is following primary.
   ./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
- Start repmgrd deomon process on master and standby to enable automatic failover.
- And verify the events for the cluster.
- Verify replication replication between servers using pg_Stat_replication view.



