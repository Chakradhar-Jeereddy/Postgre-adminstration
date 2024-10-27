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
