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
- Yum -Y install repmgr12.x86_64 ( version 5.2) on both sides
- Modify the following parameters in postgresql.conf file
- Max_wal_senders=10
- Max_replication_slots=10
- Wal_level= replica 
- Hot_standby = on
- Archive_command ='bin/true'
- Listen_addresses=’*’
- Shared_preload_libraries= 'repmgr'  -> to load the libraries
- Wal_log_hints= on
- Start Postgres on primary.
- Create user repmgr as superuser
```
createuser -s repmgr
```
- Create database repmgr with owner repmgr
```
createdb repmgr -O repmgr
```
- Configure pg_hba.conf and include repmgr
```
host repmgr repmgr 192.168.0.1/24 trust
```
- Create a repmgr file(eveything register, unregister and failover happens based on this file)
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

Note = follow_command is used to tell standby which primary to follow in case of more then one standby.
```

- Restart postgresql
```
systemctl restart postgressql-12
```
- Register primary server to repmgr
```
cd /usr/pgsql-12/bin
./repmgr -f /var/lib/pgsql/repmgr.conf primary register
- Verify whether primary server is register with repmgr
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
```
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
```
    /usr/pgsql-12/bin/repmgr -h 192.168.1.8 -U repmgr -f /var/lib/pgsql/repmgr.conf standby clone –dry-run
Note: it will fail, to fix a new user replication should be added to pg_hba.conf file.
host replication replication 192.168.1.0/24 trust
now run the dry run again
```
- Start the clone of standby server
```
     /usr/pgsql-12/bin/repmgr -h 192.168.1.8 -U repmgr -f /var/lib/pgsql/repmgr.conf standby clone
```
- Start postgresql on standby server.
```
systemctl start postgressql-12
```
- Register standby server with repmgr.
```
    cd /usr/pgsql-12/bin
    ./repmgr -f /var/lib/pgsql/repmgr.conf standby register
```
- Verify whether standby server is register with repmgr and standby is following primary.
```
   ./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
```
- Start repmgrd deomon process on master and standby to enable automatic failover.
```
repmgrd -f /var/lib/pgsql/repmgr.conf
```
- And verify the events for the cluster.
```
tail -300f /var/lib/pgsql/repmgr.log
```
- Verify replication replication between servers using pg_stat_replication view.
```
syntax: select * from pg_stat_replication;
```




