- Default location of the file is under data directory, can't modify the location.
- The file is updated when we use alter system command, should not be modified manually.
- It is the last file read by postgresql during restart and this file exists from v9.4 and above
- To use alter system command,  the account should have superuser access

- Example - 
```
  $ psql -c "ALTER SYSTEM SET work_mem TO '8MB'"
  $ pg_ctl -D $PGDATA reload
          or
  $ psql -c "select pg_reload_conf()"
          or
  $ kill -HUP <pid_of_postmaster>
  $ psql -c "show work_mem"
  ```
  - For the new value to reflect we need to reload the configuration, or restart.
 - To find pid of postgres, check it in postmaster.pid file
```
cat $PGDATA/postmaster.pid | head -1
```
- Not all the parameters can be modified without a restart.
- Some parameters, such as shared_buffers, listen_addresses, archive_mode, and so on.
- Require you to restart Postgres to get the changes into effect.
  
- Example of changing static parameter
```
    $ psql -c "ALTER SYSTEM SET shared_buffers TO '512MB'"
    $ pg_ctl -D $PGDATA reload
    $ psql -c "select name, setting, pending_restart from pg_settings where name = 'shared_buffers'"
        name           | setting | pending_restart
       ----------------+---------+-----------------
        shared_buffers | 16384   | t
    $ pg_ctl -D $PGDATA restart -mf
```
