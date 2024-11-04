- To prevent I/O bottlenecks need to keep wals and data on different
  disks(for transaction intensive database)
- This operation requires downtime
   1) Create directory and give permissions to postgres user
  ```
     mkdir -p /wals
     chown postgres:postgres /wals
     pg_ctl -D $PGDATA stop -mf
  ```
  2) Move all the existing WALs and the archive_status directory inside pg_wal
     to the new directory on another disk. Make sure that pg_wal is empty and everything is moved
     to a new directory:
```
  $ mv $PGDATA/pg_wal/* /wals
```
- Use rsync to avoid a huge downtime while copying a huge number of WAL segments to a different disk.
```
  $ pg_ctl -D $PGDATA stop -mf
  $ rsync -avzh $PGDATA/pg_wal/ /wals
``` 
- Create a symlink after removing the old WAL directory:
```
  $ rmdir $PGDATA/pg_wal
  $ ln -s /wals pg_wal
  $ ls -alrth pg_wal
    lrwxrwxrwx. 1 postgres postgres 5 Nov 10 00:16 pg_wal -> /wals
```
- Start your PostgreSQL cluster now:
```
   $ pg_ctl -D $PGDATA start
```

How it works...
To move pg_wal, we must add a new disk to the server and create the new directory that
the WAL segments will be stored in. We also need to make sure that we give appropriate
permissions to the directory, as shown in Step 1. As this requires you to shut down the
Postgres server to move the WAL directory, we could use a command similar to the one
shown in Step 1 to shut down Postgres.
If you have a huge number of WAL segments, you can avoid more downtime by skipping
this step and moving on to Step 2b. If not, you may use Step 2a, which simply moves all the
content of the existing pg_wal directory to the new directory.
If you wish to avoid the huge downtime and wish to skip Steps 1 and 2a, you could simply
use Step 2b, which uses rsync to copy all the existing WAL segments from pg_wal to the
new WAL directory. Once done, we can simply shut down Postgres and use rsync again to
copy the newly generated WAL segments. After moving all the WAL segments, remove the
old WALs directory, and create a symbolic link to the new directory, as shown in step 4.
As we have seen, pg_wal is not removed from the data directory permanently. Instead, a
symbolic link is created to the new directory that we are moving WALs to. Once the
symlink has been created, we can start PostgreSQL, as shown in Step 5, which starts writing
the newly generated WAL segments to the new WAL directory.
If you have a high availability cluster with 1 master and 1 or more standbys, you can
perform these steps in a rolling fashion or all at once. Having different locations in each of
the servers of a replication cluster is always possible. Thus, you can stop all the servers and
perform these steps or perform them on one server after another.
