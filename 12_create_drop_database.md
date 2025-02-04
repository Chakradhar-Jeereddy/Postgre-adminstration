- Use createdb and dropdb utility or it can be done from psql
- Need to be granted with superuser or createdb role or should be the owner of the database

- createdb --help
```
$ createdb --help
createdb creates a PostgreSQL database.
Usage:
 createdb [OPTION]... [DBNAME] [DESCRIPTION]
Options:
 -D, --tablespace=TABLESPACE default tablespace for the database
 -e, --echo show the commands being sent to the
server
 -E, --encoding=ENCODING encoding for the database
 -l, --locale=LOCALE locale settings for the database
 --lc-collate=LOCALE LC_COLLATE setting for the database
 --lc-ctype=LOCALE LC_CTYPE setting for the database
 -O, --owner=OWNER database user to own the new
database
 -T, --template=TEMPLATE template database to copy
 -V, --version output version information, then
exit
 -?, --help show this help, then exit
Connection options:
 -h, --host=HOSTNAME database server host or socket
directory
 -p, --port=PORT database server port
 -U, --username=USERNAME user name to connect as
 -w, --no-password never prompt for password
 -W, --password force password prompt
 --maintenance-db=DBNAME alternate maintenance database
```
- By default, a database with the same name as the current user is
   created.Report bugs to <pgsql-bugs@postgresql.org>.

- Command to create a database 
  $ createdb -e pgtest -O user1
  SELECT pg_catalog.set_config('search_path', '', false)
  CREATE DATABASE pgtest OWNER user1;

- create a database using a template
  $ createdb -e pgtest -O user1 -T percona
  SELECT pg_catalog.set_config('search_path', '', false)
  CREATE DATABASE pgtest OWNER user1 TEMPLATE pg11;

- dropdb options
```
dropdb --help
dropdb removes a PostgreSQL database.
Usage:
 dropdb [OPTION]... DBNAME
Options:
 -e, --echo show the commands being sent to the
server
 -i, --interactive prompt before deleting anything
 -V, --version output version information, then exit
 --if-exists don't report error if database doesn't
exist
 -?, --help show this help, then exit
Connection options:
 -h, --host=HOSTNAME database server host or socket
directory
 -p, --port=PORT database server port
 -U, --username=USERNAME user name to connect as
 -w, --no-password never prompt for password
 -W, --password force password prompt
 --maintenance-db=DBNAME alternate maintenance database
Report bugs to <pgsql-bugs@postgresql.org>.
```
- Command to drop database
```
$ dropdb -i pgtest
Database "pgtest" will be permanently removed.
Are you sure? (y/n) y

List the databases to be dropped 
user \l or \l+
or pg_databases

$ psql -c "select oid, datname from pg_database"
```
- to get all options
```
postgres=# \help CREATE DATABASE
Command: CREATE DATABASE
Description: create a new database
Syntax:
CREATE DATABASE name
 [ [ WITH ] [ OWNER [=] user_name ]
 [ TEMPLATE [=] template ]
 [ ENCODING [=] encoding ]
 [ LC_COLLATE [=] lc_collate ]
 [ LC_CTYPE [=] lc_ctype ]
 [ TABLESPACE [=] tablespace_name ]
 [ ALLOW_CONNECTIONS [=] allowconn ]
 [ CONNECTION LIMIT [=] connlimit ]
 [ IS_TEMPLATE [=] istemplate ] ]

Create database with a connection limit at database level
CREATE DATABASE mydb WITH OWNER user2 CONNECTION LIMIT 200;
```
postgres=# \l+ mydb

- Find the location of database in filesystem
```
psql -c "select oid, datname from pg_database"
 oid | datname
-------+-----------
 13878 | postgres
 
 ls -ld $PGDATA/base/13878
drwx------. 2 postgres postgres 8192 Sep 1 07:14
/var/lib/pgsql/13/data/base/16384
```
- Find location of table
```
$psql -d testing -c "CREATE TABLE employee (id int)"
$psql -d testing -c "select oid, relname from pg_class where relname = 'employee'"
oid | relname
-------+----------
 16385 | employee
 
 psql -d testing -c "select pg_relation_filepath('employee')"
 pg_relation_filepath
----------------------
 base/16384/16385
 ```
A file with the name 16385 (the same as the oid of the employee table) is created 
inside the 16384 directory. The directory 16384 corresponds to the oid of the database we created. 
So, this indicates that a table in PostgreSQL is created as a file inside the database 
directory inside the base directory. This base directory is, of course, one of the subdirectories 
of the data directory.






 







