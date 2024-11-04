- User defined tablespace to distribute objects across muliple disks.
- Steps
   1) create directory
   2) If creating tablespace in master-slave replication cluster, we should make sure tablespace exists on standby also.
   3) Before restoring backup, we should create respective tablespace directories

- Commands
```
sudo mkdir -p /newtablespace
sudo chown postgres:postgres /newtablespace
$ psql -c "CREATE TABLESPACE newtblspc LOCATION '/newtablespace'"
```
- Check the pg_tblspc directory to see a symlink to the new tablespace
```
ls -l $PGDATA/pg_tblspc
 total 0
 lrwxrwxrwx. 1 postgres postgres 14 Nov 3 00:24 24611 -> /newtablespace
```
- There will be many such entries when we create more tablespaces. 
- To list tablespaces
```
postgres=# \db
List of tablespaces
 Name | Owner | Location
 ------------+----------+----------------
 newtblspc | postgres | /newtablespace
- Create a table in new tablespace
$psql -c "create table chakra(id int) tablespace newtblspc;
```
- TO find the location of the table
```
$psql -c "select pg_relation_filepath('chakra');
 pg_relation_filepath
 ---------------------------------------------
 pg_tblspc/24611/PG_13_201909212/14187/24612

 \d employee
 Table "public.employee"
 Column | Type | Collation | Nullable | Default
 --------+---------+-----------+----------+---------
 id | integer | | |Tablespace: "newtblspc"
 ```
- Moving tables to a different tablespaces
- Cause downtime as it needs exclusive lock
- can be done using alter table or pg_repack command
 ```
 ALTER TABLE percona.foo SET TABLESPACE newtblspc;
 ALTER INDEX percona.foo_id_idx SET TABLESPACE newtblspc;
 ALTER TABLE <schemaname.tablename> SET TABLESPACE <tablespace_name>;
 ALTER INDEX <schemaname.indexname> SET TABLESPACE <tablespace_name>;
```
 
 
