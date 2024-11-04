- Install the Postgres client
```
yum install postgresql13
```

- Local connection
 $psql

- Remote connection
 ```
 $psql -h Server_A_IP -p 5432 -d postgres -U postgres
 $psql -d percona -c "select * from foo.employee" > results.out
```
- Shortcuts
```
 \l           -list databases
 \l+          -list databases and their sizes
 \c percona   -Switch to another database, select current_database();
 \dn          -list of schemas in database
 \dn+         -list of schemas with privileges in a database
 \dt          -list of tables in a database
 \dt+         -list of tables and size under user or public
```
```
  percona=# show search_path ;
   search_path
   -----------------
  "$user", public
  \dt foo.*   -under a specific schema
  \dt+ foo.*
  \d foo.employee       - Describing a table
  \?          -to get help
```  
 - To see SQLs that run in background when a shortcut command is used
```
   psql -d percona -E
   \dn
    ********* QUERY **********
    SELECT n.nspname AS "Name",
    pg_catalog.pg_get_userbyid(n.nspowner) AS "Owner"
    FROM pg_catalog.pg_namespace n
    WHERE n.nspname !~ '^pg_' AND n.nspname <> 'information_schema'
    ORDER BY 1
   ```
