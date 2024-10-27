```
-- Work_mem(4MB) is the memory area used by query for operations such as JOINS, OREDER BY, DISTINCT, hash joins and merge joins.
-- Large sort operations which doesn't fit in the small work_mem offen gets spilled into the disk and temporary files are created during execution.
-- Temp files are created in data directory ($PGDATA/base/pgsql_tmp) if you have huge tempfile it will fill up data directory
-- Temp files are only kept for the duration of a query. Once the query finishes or cancelled, the temp files are cleand up.
-- Temp_tablespaces specifies the tablespace in which temporary objects created (temp tables and indexes on temp tables)
-- When a create command does not explicitly specify a tablespace.
-- Temporary files for purposes such as sorting large data sets are also created in these tablespaces.
-- The default value is an empty string, which results in all temporary objects being created in the default tablespace of the current database.
```
```
-- Move TEMP Files/Tables From Default Location to New Location:
-- For source code installation, the logging is by default not enabled.
-- open configuration file postgresql.conf enable logging
logging_collector = on
log_directory = 'log'

create temporary table test1 ( empno int);
-- To drop "drop table test1;"

select pg_relation_filepath('test1');
 pg_relation_filepath
----------------------
 base/5/t3_16390

-- New location for temp tables (created in SQL will be available during session)
mkdir -p /u05/app/16.2/Temp_files 
-- Create new temp1 tablespace
create tablespace temp1 location '/u05/app/16.2/temp_files';
--Update default temp tablespace
alter system set temp_tablespaces = 'temp1';
show temp_tablespaces; => Default location is ../data/base

-- Reload the config file
select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

-- show temp_tablespaces;
 temp_tablespaces
------------------
 temp1
-- create new temp table and check its location
create temporary table test2 ( empno int);
 select pg_relation_filepath('test2');
            pg_relation_filepath
--------------------------------------------
 pg_tblspc/16396/PG_16_202307071/5/t3_16397
(1 row)

-- For user defined tablespaces a soft link is created under pg_tblspc
cd /u01/16.2/data/pg_tblspc
ls -ltr
lrwxrwxrwx 1 postgres postgres 20 Oct 26 20:34 16396 -> /u01/16.2/temp_files (shows only when session is active)

```
