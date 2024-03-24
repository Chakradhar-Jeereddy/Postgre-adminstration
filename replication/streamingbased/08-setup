# Steps
## On master
- Check following parameters in postgresql.conf
   - listen_address - *   (change requires restart)
   - wal_level - replica  (minimal, replica, or logical and change requires restart)
   - host_standby - On     change requires restart)
- Restart the cluster
```
Create repuser with replication role and encypted password 'xxxx';
psql> create user repuser with replication encrypted password 'abc';
CREATE ROLE
psql> \du
Role name   | List of role attributes | Member of
repuser     | replication             | {}
```

- Modify pg_hba.conf and specify the IP address of the both master and standby with md5 method.
```
Standby connects to the master to fetch the wal records
vi pg_hba.conf
host  replication  repuser  192.168.1.0/24  md5
:wq
```
- Reload/restart the configuration.

## On Standby:
- Stop database
- Delete all the files under data directory (take backup of postgresql.conf)
- Run pg_basebackup on to clone the standby database
```
pg_basebackup --help
pg_basebackup -h <master-ip> -U repuser -p 5432 -D $PGDATA -Fp -Xs -P -R -C -S pgstandby
  -h = host
  -U = user
  -p = port
  -D = data directory
  -F = format(plain or tar)
    -p = plain
  -X = wall method(none||fetch||stream)
   -s = stream
  -P = postgres
  -R = write configuration for replication
 - C = creation of replication slot named by the -S option in master.
```
- Options -R, -C, -S automatically create standby.signal and postgresql.auto.conf file in directory $PGDATA.
- It also creates a replication slot in master server.
```
cat postgresql.auto.conf (it adds connection information of master)
Primay_conninfo = 'user=repuser password=abc host=192.168.1.8 port=5432 slsmode=prefer primary_slot_name = 'pgstandby'
```
- Start standby database
```
Content of the log
database system is ready to accept connections
entering standby mode
redo starts at 0/214422142
consistent recovery state reached at 0/214422356
database system it ready to accept read only connections
started streaming WAL from primary at 0/30000000 on timeline 1
````

## Test the replication
- create table and insert rows on master and check replication slot
```
psql> create table test(id int);
psql> insert into test values(100);
psql> select * from pg_replication_slots;
```
- Verify the data in standby and try to insert data on standby
```
psql> selet * from test;
psql> insert into test values(200);
ERROR: Cannot execute insert in a read-only transaction
```

  

