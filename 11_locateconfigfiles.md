```
/psql -c "show config_file"
  or 
ps aux | grep /postgres
/usr/pgsql-12/bin/postgres -D /var/lib/pgsql/13/data --configfile=/etc/postgresql/13/main/postgresql.conf
  or
select name , setting from pg_settings where setting ilike '%.conf';
  config_file | /etc/postgresql/13/main/postgresql.conf
  hba_file | /var/lib/pgsql/13/data/pg_hba.conf
  ident_file | /var/lib/pgsql/13/data/pg_ident.conf
```
 
 - Steps to change default location of postgresql.conf
```
   pg_ctl -D $PGDATA stop -mf
   $ sudo mkdir -p /pgconfigs
   $ sudo chown postgres:postgres /pgconfigs
   $ sudo chmod -R 700 /pgconfigs
   $ mv $PGDATA/postgresql.conf /pgconfigs/postgresql.conf
   $ pg_ctl -D $PGDATA -o '--config-file=/pgconfigs/postgresql.conf' start
   $ psql -c "show config_file"
      config_file
      ----------------------------
      /pgconfigs/postgresql.conf
```    
- How tos

- Modifying the postgresql.conf file's location is very easy and straightforward to do. To
  modify the location of the configuration file, we must shut down the PostgreSQL server, as
  shown in Step 1. Once the PostgreSQL cluster is stopped, we must create the directory that
  the configuration file must be stored in as shown in Step 2. We must also ensure that we
  give appropriate ownership and privileges to Postgres users on that directory and file, also
  shown in Step 2. You may replace /pgconfigs in Step 2 with an appropriate directory
  name.

- Now, we can simply move or copy the file to the new location, as shown in Step 3. Since the
  location of the configuration file has now changed from its default location, we must start
  PostgreSQL using an additional flag that reads the specified configuration file, as shown in
  Step 4. Once we have started PostgreSQL, we can validate it using the show command, as shown
  in Step 5.
