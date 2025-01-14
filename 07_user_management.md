```
create role  - no login
create user  - with login (connect privilege)
CREATE USER percuser WITH ENCRYPTED PASSWORD 'secret';
or
CREATE ROLE percuser WITH LOGIN ENCRYPTED PASSWORD 'secret';
or 
CREATE ROLE percuser;
ALTER ROLE percuser WITH LOGIN ENCRYPTED PASSWORD 'secret';
```
- Verify if user can connect -
```
select rolcanlogin from pg_roles where rolname = 'percuser';
```
- To list the users or roles we could either query pg_roles or \du
```
psql -c "\du"
 List of roles
 Role name | Attributes | Member of
 ------------------+-------------------------------------------------------
-----+-----------
 app_user | | {}
 dev_user | | {}
 percuser | | {}
 postgres | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 read_only_scott | Cannot login | {}
 read_write_scott | Cannot login | {}

psql -c "SELECT rolname, rolcanlogin FROM pg_roles where rolname NOT LIKE 'pg_%'"
rolname | rolcanlogin
 ------------------+-------------
 postgres | t
 percuser | t
```
- Dropping a user
  There will also be a great challenge when a user who is being dropped owns one or more objects. In this
  case, we may have to re-assign the ownership to another user before dropping the user
  without dropping the objects it owns. In this recipe, we shall see how a user can be dropped
  safely and the best practices you can follow to avoid dropping the objects a user owns. 
```
postgres=# DROP USER percuser;
 ERROR: role "percuser" cannot be dropped because some objects depend on it
 DETAIL: privileges for database percona
 2 objects in database pmm
 2 objects in database percona
```
- Re-assign ownership on objects and revoke permissions from user being dropped.
```
 $ psql -U postgres -d percona -c "REASSIGN OWNED by percuser TO pmmuser"
 $ psql -U postgres -d pmm -c "REASSIGN OWNED by percuser TO pmmuser"
 postgres=# REVOKE ALL PRIVILEGES ON DATABASE percona FROM percuser;
 REVOKE
 postgres=# REVOKE ALL PRIVILEGES ON DATABASE pmm FROM percuser;
 REVOKE
 Drop the user after the revocation is successful:
 postgres=# DROP USER percuser ;
 DROP ROLE
```
- Commands to grant and revoke privileges
 ```
 GRANT SELECT ON employee TO percuser;
 REVOKE SELECT ON employee FROM percuser;
 GRANT ALL ON employee TO percuser;
 GRANT ALL ON employee TO percuser;
 GRANT ALL ON DATABASE percona TO percuser ;
 REVOKE ALL ON DATABASE percona FROM percuser ;
 ```
- Role management
```
Create the read-only and read-write roles for each schema respectively:
CREATE ROLE scott_readonly;
CREATE ROLE scott_readwrite;
CREATE ROLE tiger_readonly;
CREATE ROLE tiger_readwrite;
GRANT USAGE, SELECT ON ALL TABLES IN SCHEMA scott TO scott_readonly;
GRANT USAGE, SELECT ON ALL TABLES IN SCHEMA tiger TO tiger_readonly;
GRANT USAGE, SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA tiger TO tiger_readwrite;
GRANT USAGE, SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA scott TO scott_readwrite;
GRANT scott_readwrite to appuser1;
GRANT tiger_readwrite to appuser2;
GRANT scott_readonly to devuser1;
GRANT tiger_readonly to devuser2;
```
 
```
Command:     REASSIGN OWNED
Description: change the ownership of database objects owned by a database role
Syntax:
REASSIGN OWNED BY { old_role | CURRENT_ROLE | CURRENT_USER | SESSION_USER } [, ...]
               TO { new_role | CURRENT_ROLE | CURRENT_USER | SESSION_USER }

```

```
Password managment:
select * from pg_shadow;
alter system set password_encryption='md5' or 'scram-sha-256'
alter user nihu with password 'hellome';
The password gets encypted and stared in system tablespace
```



