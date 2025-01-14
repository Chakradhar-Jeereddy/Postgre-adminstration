```
Microsegmentation
Network security
Authentication security
Instance security
database security
schema security
table security
column security
row security
```

```
Microsegmentation - Only IPs from specific networks are allowed.
Network security - Binding addess/listen_addresses
By defaut postgresql listens to localhost and doesn't accept or reject remote connections because it doesn't
listen on port for remote connections. the error comes from server when we do telnet.
We can set the bind address to '*" all ips assing to server and not client ips.
It can only listen on port.
Max_connections defaults to 100, 3 for superuser and 97 for regular user or 100 superusers
show superuser_reserved_connections;
The other network parameter are unix_socket_%
show unix_socket_directories;
Defaults to /tmp
show unix_socket_permissions;
should be 0777
show unix_socket_group;
The socket file s.pgsql.5432.lock contains proccess id of postmaster, directory, port and socket directory.
The unix socket is used by local connections when auth method is used as peer.

Authentication security - pg_dba.conf
type - local, host, hostssl, hostnossl,replication
Database
User
IP
Method -
 Peer - if OS user and db user is same then it uses connects, you already os authenticated.
 trust - no password is supplied from client
  mdf - password enctryption
  reject - reject incomming connections
  ldap - external authentication, username and password should be registered in ldap server and sssd client shloud be running in
         postgres. Same user should exist in database, when it tries connect, the supplied username and password are verfied
         in ladp server, if matches it allows to connect. So external authentication.
         To contact ldap server we specify the ldap server, port, tlsversion, userprefix, userssufix and ldap users in pga_dba.conf
         users - +ldap_read,+ldap_readwrite,+ldap_readwritefull,+ldap_admin,+ldap_dba,ldap_execute
         Method - ldap,ldapserver, ldapport, ldaptls='1' ldapprifix="uid", ldapsufix=" dn=......"
   scram-sha-256
   ident
In my company
local all postgres         peer
local all walle            peer
host  all postgres 0.0.0.0/0 scram-sha-256
host  all telegraf  IP       scram-sha-256   
host  all repmgr    primary  scham-sha-256
host  all gdat       ip       scham-sha-256
host  all repmgr     standby   scham-sha-256
hostssl all ldap,,,   ip       ldap

Instance level secutiry
Users or roles are created at instance level, they are visible from all databases, however they connect to database to which
they got access. The user previleges can be found from \create user. The role doesn't get login.
Privilegrs are
login/nologin
createdb/nocreatedb
createrole/nocreattole
inherit/noinherit
valid until timesamp
replication/no replication can stream wal
bypassrls/nobypassrls
connection limit connlimit
sysid uid  - create user with specific id
encrypt password

User who creats database will be the owner of the databases unless specfied
create database with owner ...
user can access database as he is public, to restric user and grant permissions only to required db
revoke all on database from public;
grant connect to database;

Database level security
 permissions are create and connect
 create is to create schema and connect is to enter into database

Schema level security
create is to create table and usage to enter into schema
user by default can create table in public schema.
revoke all on public from public; from v15 only owne of the database can create table in public schema

table level security
create table can create and drop table
grant select, update,delete, truncate(exlusive lock),inheritance,trigger on table to user.
arwdDxt
select * from aclexplod({postgres=arwdDxt/postgres}')
\z table name to see table level permissions

column level security
Grant permissions on specific columns
grant select(id) on table to user;
select * from table will no longer work.

Rowlevel security
We can filter rows based on condition
enable RLS for table.
alter table tabname enable row level security;
\d tabname to check if its enabled.
A default policy is applied if the RLS is enabled and it will deny all rows.
User will get zero rows unless he is super user.
to read delete update rows create policy with using clause
create policy polname as {permissive|restrictive} on table for select to user usnig(column>10);
select * from table;
Permissive is OR and Restrictive is AND when more then one policy exists for table.
For insert use with check clause

create policy plname as restrictive on table for insert with check(id=10);
Check policy details
select * from pg_polcies;

drop policy if exists palname on table;
alter table tabname disable row level security;
```

```
Configuring default privileges
alter default privilege for ROLE | USER in schema grant....;
Applies to newly created objects.
```
         
  







