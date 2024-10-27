```
* Tablespace is basically a directory in postgresql which contains data files.
* Postgesql cluster has two tablespaces (pg_default and pg_global)
* pg_default is a base subdirectory and pg_global is global subdirectory
* All user objects are stored under Base/pg_default tablespace
* All catalog tables and its indexes which are used across the cluster are stored in pg_global.
* default_tablespace parameter sets the tablespace to be used by default, in which the database objects gets created.
* By defaut, it has empty string value, which means all objects created under pg_default tablespace.
* If we specify some value in the parameter and it turns out that such tbs doesn't exist, there wont be error.
* Objects gets created under pg_default tablespace.
```

```
show default_tablespace;
 default_tablespace
--------------------

select  pg_relation_filepath('test1');
 pg_relation_filepath
----------------------
 base/5/16413
```

```
-- Scenarios
1) Already have PostgreSQL installed and running and want to ensure all my future objects are created in new tablespace.
a) Create a tablespace on new location. Syntax - create tablespace <tablespace name> location <'/path'>;
b) Change the default tablespace location parameter in postgresql.conf to the tablespace name restart cluster.
c) Move old objects from pg_default to respective tablespaces.

2) I have just installed Postgresql and yet to create a database.
  Create a new tablespace and make it default in postgresql.conf and restart the cluster.
  All objects goes into the default tablespace from the start.
 -- Create database sales owner salesapp tablespace sales; (Good for multiple databases).

3) Already have multiple databases storing objects on pg_default.
   I want to have different tablespace for each database.
   a) Don't disturb default_tablespace parameter in postgresql.conf
   b) Create tablespace in new location.
   c) Alter database <dbname> set tablespace <tablespace name> (do this for each database).
   d) Move old objects from pg_default to respective tablespaces.
```

```
-- Identify objects in the default tablespace

select tablename from pg_tables where tablespace is null and schemaname='public';
select indexname from pg_indexes where tablespace is null and schemaname='public';

-- Move tables and indexes to new tablespaces as superuser
-- Downtime consideration: Moving large tables might temporarily lock them.
-- If you need minimal downtime, plan for low-traffic times or consider moving objects in batches.
alter table public.table_name set tablespace new_tablespace;
alter index public.index_name set tablespace new_tablespace;

-- To automate using dynamic sql

select 'alter table '|| schemaname || '.'|| tablename ||' set tablespace new_tablespace;'  from pg_tables
where tablespace is null and schemaname='public';

select 'alter table '|| schemaname || '.'|| indexname ||' set tablespace new_tablespace;'  from pg_indexes
where tablespace is null and schemaname='public';


```









