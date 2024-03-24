## Objective: Setup log based shipping in Linux.

- Initial setup on Master server:
    - Create ssh key on master and copy it to standby
      - Syntax : ssh-keygen
      - Copy the ssh key to standby server
```
Syntax : ssh-copy-id  postgres@ipadress
```
- Test copying a file from master to standby without entering password
- Shutdown Master database 
- Modify the content of postgres.conf file and edit parameters archive_on, archive_command and archive_timeout.
```
Archive_mode: on
archive_timeout: 60
Archive_command: rsync -a %p postgres@ipaddress:/location of archive folder
Note - Setting timeout for archiving on master having less workload can send empty files to stanby.
```

- Initial Setup Standby Database:
- Create a archive directory on standby which is accessible by Master user.
- Shutdown postgresql on standby
- Delete all the contents of /Data directory

- Steps:
  - Startup postgresql on Master database
```
systemctl start postgresql-14
systemctl status postgresql-14
```
- Connect to the database using psql shell
- Issue connect pg_start_backup
```
Syntax : select pg_start_backup(‘dbrep’);
pg_start_backup
------------------
0/A1000060
dbrep is a label
```
- Copy Data directory for master to standby.
```
Syntax : rsync -avz /var/lib/pgsql/12/data/* postgres@192.168.1.9:/var/lib/pgsql/12/data/
```
- End backup on master.
```
Syntax : select pg_stop_backup();
Notice: All required WAL segments have been archived.
        ps_stop_backup
        0/A2000050
```
- Comment out archive_on, archive_command and archive_timeout parameters in postgresql.conf in standby server.
- Setup recovery command in postgresql.conf
```
#archive_on:
#archive_command:
#archive_timeout:
Restore_command = ‘cp /var/lib/pgsql/12/archive/%f %p’
Create a standby.signal file in data directory.
touch standby.signal
Note: Make sure archives are being copied to standby
ls -ltr /var/lib/pgsql/12/archive/
```
- Start postgres on standby server
- Check the log files whether the postgres has entered standby mode and accepting read only connections.
```
database system ready to accept connections
entering standby mode
restored log file "0000000A1" from archive
consistent recovery state reached at 0/A2000050
database is ready to accept read only connections
```
- Test Standby Database:
- Try to create table and insert few records on master and check whether they are populated On the standby . 
- try to create a table on standby or delete any records on standby
```
psql(master)
psql>\l
psql> create table master(id int);
psql> insert into master values(1);
psql(standby)
psql> select * from master;
```
