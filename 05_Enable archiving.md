- Cluster restart is required to enabl archiving mode.
```
$ psql -c "show archive_mode"
$ psql -c "ALTER SYSTEM SET archive_mode TO 'ON'"
$ pg_ctl -D $PGDATA restart -mf
$ mkdir -p /backups/archive
$ chown postgres:postgres /backups/archive
$ psql -c "ALTER SYSTEM SET archive_command TO 'cp %p /backups/archive/%f'"
$ psql -c "select pg_reload_conf()"
```
- How tos

- To enable WAL segments to be archived to permanent storage, follow these steps:
   1. Set the archive_mode parameter to ON (requires restart).
   2. Set the archive_command parameter to a shell command or a script that can send archives to an archive location.

- Before setting the archiving to a location, we must validate whether archive_mode is set to ON, as shown in Step 1. 
- If it is not set to ON, we can set it to ON using the command shown in Step 2. 
- As a change to archive_mode requires a restart, we need to restart the
  PostgreSQL cluster using a command similar to what we saw in Step 3.

- To copy WAL segments to a different location for archiving, we need to provide the appropriate shell command. 
- Which can be used to perform the copy. In my setup, I am trying to safely archive the WAL segment to a NAS mounted named /backups. 
- So, I could just let the archiver process issue a simple copy command to copy the WAL segment to the NAS mount point, as shown in Step 5.

- These two values are substituted by Postgres.
- Here, %p => refers to the path of the WAL segment to be archived, 
- while %f => refers to the WAL segment's name.

- Now, to get the changes to archive_command into effect, we must perform either a reload or a SIGHUP, as seen in Step 5. 
- Once done, archiving will be in effect.
- There's more...
- As an example, after setting archive_command with the previously mentioned shell command, 
  the command that's executed by Postgres to archive one of the WAL segments may look as follows:
```
cp /var/lib/pgsql/12/data/pg_wal/00000001000000D2000000C9  /backups/archive/00000001000000D2000000C9
As you can see, it copies the WAL segment to the location specified in the shell command.
select pg_switch_wal();
```

