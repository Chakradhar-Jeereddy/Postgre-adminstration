Check table and index size in postgresql

We can create tables without privileges under public schema but on other schemas we need permissions (read/write)

Table size
percona=# \dt+ foo.dept
 List of relations
 Schema | Name | Type | Owner | Size | Description
--------+------+-------+----------+---------+-------------
 foo | dept | table | postgres | 3568 kB |


To match a pattern
percona=# \dt+ public.*bench*

percona=# \di+ foo.dept_pkey
 List of relations
 Schema | Name | Type | Owner | Table | Size | Description
--------+-----------+-------+----------+-------+---------+---------
----
 foo | dept_pkey | index | postgres | dept | 2208 kB |


We will use pg_relation_size to get the table and index size: 
 percona=# select pg_relation_size('foo.dept');
 pg_relation_size
------------------
 3629056
 
 After connecting appropreate database user \dt+ name or \di+ name to get size of tables and indexes
 Or use pg_relation_size() function, supply the object name in it.
 
 Size of all tables of a schema
 postgres=# SELECT schemaname, relname,
pg_size_pretty(pg_relation_size(schemaname||'.'||relname)) as pretty_size
FROM pg_stat_user_tables where schemaname = 'foo' and relname IN
('employee','sales','bar');


 
