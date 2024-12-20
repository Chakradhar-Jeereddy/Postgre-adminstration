- Default schema is public, all tables can be created in it.
- For better managment, we can use different schemas, objects with same name can exist in different schemas
- When we create schema,it doesn't create user with same name as it does in oracle. Need to be created saparetly
- Connect to database 
```
psql -d percona
Create schemas
CREATE SCHEMA foo;
CREATE SCHEMA IF NOT EXISTS foo;
NOTICE: schema "foo" already exists, skipping
CREATE SCHEMA
```
- By default the user who ran create schema command will be the owner of the schemas, to change ownership use authorization command.
```
select current_user;
percona=# CREATE SCHEMA foo;
CREATE SCHEMA
percona=#
percona=# SELECT n.nspname AS "Name",
 pg_catalog.pg_get_userbyid(n.nspowner) AS "Owner"
FROM pg_catalog.pg_namespace n
WHERE n.nspname = 'foo';
 Name | Owner
------+----------
 foo | postgres
(1 row)
```
- We can grant create schema privilege to non-superuser to create schema after connecting to that user
```
$ psql -d percona -U user1
percona=> select current_user;
 current_user
--------------
 user1
(1 row)
percona=> CREATE SCHEMA foo;
CREATE SCHEMA
percona=> SELECT n.nspname AS "Name",
 pg_catalog.pg_get_userbyid(n.nspowner) AS "Owner"
FROM pg_catalog.pg_namespace n
WHERE n.nspname = 'foo';
 Name | Owner
------+-------
 foo | user1
(1 row)
```
- As a super user, we can schange the owner of the schema
```
percona=# select current_user;
 current_user
--------------
 postgres
(1 row)
percona=# CREATE SCHEMA foo AUTHORIZATION user1;
CREATE SCHEMA
percona=#
percona=# SELECT n.nspname AS "Name",
 pg_catalog.pg_get_userbyid(n.nspowner) AS "Owner"
FROM pg_catalog.pg_namespace n
WHERE n.nspname = 'foo';
 Name | Owner
------+-------
 foo | user1
(1 row)
```

- When you use AUTHORIZATION to give ownership of a schema to a non-superuser (user2)
  as a non-superuser (user1), the user executing CREATE SCHEMA (user1 here) should be a
  member of the role (user2) who should own the schema. In the following log, you can see
  an error when user1 is giving authorization to user2 without being a member of user2:
```
percona=> CREATE SCHEMA foo AUTHORIZATION user2;
ERROR: must be member of role "user2"
postgres=# GRANT user2 to user1;
GRANT ROLE
percona=> CREATE SCHEMA foo AUTHORIZATION user2;
CREATE SCHEMA
```



