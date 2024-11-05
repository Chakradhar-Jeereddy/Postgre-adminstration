- pg_dump/pg_restore for logical backup/restore 
- pg_basebackup for physical backups
- Other open source backup tools
  - pgBackrest, Barman or wal-g as alternatives.

- Advantages of these backups tools
   - Parallel backup and restore
   - Incremental backups
   - Differential backups
   - Features for building standby replicas
   - The ability to stream backups to another server

- These file will explain the following activites
   - Backing up and restoring a database using pg_dump and pg_restore
   - Backing up and restoring one or more tables using pg_dump and pg_restore
   - Backing up and restoring globals or an entire cluster using pg_dumpall and psql
   - Parallel backup and restore using pg_dump and pg_restore
   - Backing up a database cluster using pg_basebackup
   - Restoring a backup taken using pg basebackup
   - Installing pgBackRest on CentOS/Red Hat OS
   - Installing pgBackRest on Ubuntu/Debian OS
   - Backing up a database cluster using pgBackRest
   - Restoring a backup taken using pgBackRes

- Restoring backups from pg 9.6 to pg13, use the binaries of pg13 to perform logical backup of database in v9.6 and restore it in pg13

- pg_backup format types (plain text and custom format)
   - for plain text use psql to restore 
   - for custom format use pg_restore

- pg_dump can help take the consitent backup without causing any waits to the usual database trafic.

- Limitations of pg_dump- 
   - It can't be used to take multiple databases at same time,one database backup at a time.
   - These backups are not useful for PITR,it is not possible to replay transaction logs after restoring such backups.
- To run the backups from remote machines, it should have postgresql13 package can be insttalled using
   - sudo yum install postgresql13
    - wget pgdg-redhat-repo-latest.noarch.rpm, which is available at https://www.postgresql.org/download/linux/redhat/ for Red Hat and CentOS. 
 
- for remote backup we need to use hostname,port and username parameters
   - -F -format
   - -f location of backup
   - pg_dump -h <source_server> -p <source_port> -U <username> database_name -F<format> -f <backup_location>/<dumpfile_name>.dmp


- Command to take custom backup - 
   - pg_dump -h <source_server> -p <source_port> -U <username> percdb -Fc -f /backup_dir/percdb.dmp
 
- Restore the dump into newly created database on target server, the name of the database need not be same
    - psql -c "CREATE DATABASE mypgdb"
 
- Restore all objects chakra database into mypgdb
    - pg_restore -h <target_server> -p <target_port> -U <username> mypgdb /backup_dir/percdb.dmp
 
 
- 1b) command to take backup in plain text
   - pg_dump -h <source_server> -p <source_port> -U <username> percdb -Fp -f /backup_dir/percdb.dmp
   - psql -c "CREATE DATABASE mypgdb"
   - restore using psql
   - psql -h <source_server> -p <source_port> -U <username> mypgdb -f /backup_dir/percdb.dmp
   - Note- backup must be copied to target server using scp, rsync if it was taken locally on source.

   - Backup and restore one or more tables using pg_dump and pg_restore 
   - We need to use additional argument "-t" to specify the tables to backup from a database at a time.

- Two tables account.infra and public.pgbench_history in custom format
   - pg_dump percdb -t accounts.infra -t public.pgbench_history -Fc -f /backup_dir/tables.dmp

- In plain text format as below
   - pg_dump percdb -t accounts.infra -t public.pgbench_history -Fp -f /backup_dir/tables.dmp

- Restore steps
   - psql -c "CREATE DATABASE mypgdb"
   - pg_restore -h <target_server> -p <target_port> -U <username> -d mypgdb -t accounts.infra -t public.pgbench_history /backup_dir/tables.dmp
   - psql -h <target_server> -p <target_port> -U <username> -d mypgdb -f /backup_dir/tables.dmp

-To backup muliple tables, we can specify the pattern 
   - pg_dump percdb -t accounts.* -Fp -f /backup_dir/accounts.dmp

   - Note - Only pg_restore can use -t option to selectively restore the tables from the dump. So it is always good to take the backup in non-plain text format.

- pg_dumpall, this is used to dump all databases into a single file and take backup of globals
   - For cluster and global dumps

- cluster dump
   pg_dumpall -h <source_server> -p <source_port> -U <superuser> -f /backup_dir/dumpall.sql
- globals only (users, roles and tablespaces)
   pg_dumpall -h <source_server> -p <source_port> -U <superuser> -g -f /backup_dir/dumpall.sql

- Note - pg_dumpall only supports plain text format and it should always restored with psql and parallel option can't be used.

- Using parallel option of pg_dump and pg_restore

   - create directory /bases/pg_backup
   - Use the -j option to define number of cores
   - Multiple custom format dumps created in compressed mode for each table and blob.
   - Only pg_restore can be used to restore the large databases

    - mkdir -p /backup_dir/percdb
    - pg_dump -Fd percdb -j 8 -f /backup_dir/percdb
    - pg_restore -d percdb -j 4 /backup_dir/percdb
 
     - Physical backups
     - pg_basebackup is an inbuil utility, can be used to perform PITR recovery.

-Benifits -
    - We can perform the backup on standby(offloading) which will always be in sync with prod on a faster network
    - pg_basebackup can't run in parallel mode and doesn't support incremental and differential backup, 
      upto gigabytes of cluster is ok.
 
- Requirments to run pg_basebackup
   - Empty directory 
   - As it runs backup using replication protocal, user must have replication role and must connect to server using relplication protocal.
 
- Commands - 
    - sudo mkdir -p /backup_dir/`date +"%Y%m%d"`
    - sudo chown -R postgres:postgres /backup_dir/`date +"%Y%m%d"`
    - psql -U postgres -c "CREATE USER backup_user WITH REPLICATION PASSWORD 'secret'"
    - echo "host replication backup_user <backup_server_hostname>/32 md5" >> $PGDATA/pg_hba.conf
   - $ psql -c "select pg_reload_conf()"

- backup - 
   - pg_basebackup -h <hostname> -U <user> -p <port> -D <target_directory> -c <checkpoint_mode> -F<format> -P -X<wal_method> -l <backup_label>
   - pg_basebackup -h 192.168.90.70 -U backup_user -p 5432 -D /backup_dir/`date +"%Y%m%d"` -c fast -Ft -z -P -Xs -l backup_label

- explain the options
   - The backup can be taken in plain , tar or tarzip format (Ft, Ftz) -Ft (tar format), -Fp (plain format)
   - THe backup waits for checkpoint to complete, we can use "-c fast"  fastboot instead of waiting 
   - To display the progress use -P option
   - Backup needs to be performed online, in order to make the backup consitent it also needs the wals that are generated during backup, we can either stream the wals
   - using parallel stream (-Xs) or fetch the wal segments after backup is completed.
   - Give a lable to the backup , we could use -l.

- Restoring backup -
   - For creating a standby server in replication
   - Recovering a database to a point in time

- Steps 
   - sudo mkdir -p /pgdata
   - tar xzf /backup_location/base.tar.gz -C /pgdata
   - ls -l /backup_dir/20201027/
   - Note - If the database contains one or more tablespaces outside data directory, they are backedup into a file with table's oid.

 total 16816
 -rw-------. 1 postgres postgres 1006685 Oct 27 11:48 16575.tar.gz  - tbs
 -rw-------. 1 postgres postgres 1006657 Oct 27 11:48 16576.tar.gz  - tbs
 -rw-------. 1 postgres postgres 15183012 Oct 27 11:48 base.tar.gz  - this will contain the tablespace_map file
 -rw-------. 1 postgres postgres 17094 Oct 27 11:48 pg_wal.tar.gz   - should go to pg_wal directory
 
- Each of these tablespaces must be extracted into the directories mentioned on the tablespace_map directory 
    - cat /pgdata/tablespace_map
    - 16575 /data_tblspc
    - 16576 /index_tblspc

- Extract the WAL segments generated during the backup to the pg_wal directory:
   - $ tar xzf pg_wal.tar.gz -C /pgdata/pg_wal

- Start the postgresql using the data directory 
   - $ pg_ctl -D /pgdata start

- As seen in step 3, we use the tablespace mapping to extract the tablespace contents to the
 same locations. If the locations need to be modified, it can be done by just modifying the
 tablespace_map file manually and adding the new locations for each tablespace.

   - $ tar xzf 16575.tar.gz -C /data_tblspc
   - $ tar xzf 16576.tar.gz -C /index_tblspc
   - $ tar xzf pg_wal.tar.gz -C /pgdata/pg_wal
   - $ pg_ctl -D /pgdata start






 









 








 
 
 
 










