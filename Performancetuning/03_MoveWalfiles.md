### Move Wal_files from $PGDATA to New Location

-- Create Postgresql Directory 
```
mkdir -p /u03/app/16.2/wal_files 
```
-- Change ownership to postgres
```
chown -h postgres:postgres /u03/app/16.2/wal_files

-- Stop postgresql
```
./pg_ctl stop

```
-- rsync all files from $PGDATA/pg_wal to new location
```
rsync -av /u02/app/16.2/data/pg_wal/* /u03/app/16.2/wal_files

```
-- Check all files are synced
```
ls -la  /u03/app/16.2/wal_files

-- Take a backup of pg_wal folder
```
mv /u02/app/16.2/data/pg_wal /u02/app/16.2/data/pg_wal-backup
```

-- Create a Symbolic link 
```
sudo ln -s /u03/app/16.2/wal_files/ /u02/app/16.2/data/pg_wal
```


-- Start Postgresql
```
./pg_ctl start
```

-- Verify DB connection using your db credentials/information
```
psql -h localhost -U postgres -p 5432
select pg_switch_Wal() ( check wal files in new location)
```

-- Remove the old folder
``` 
rm -rf /u02/app/16.2/data/pg_wal-backup
```
