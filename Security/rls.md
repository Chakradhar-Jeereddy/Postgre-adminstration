select * from aclexplode('{postgres=arwdDxt/postgres}');

row level security is assing to table.

alter table table_name enable row level security;
It will apply default policy which is denay. User will see empty rows.

create policy pol_name as permissive|restrixtive on table_name for select to user using(id=10);
create policy pol_name as permissive|restrixtive on table_name for insert to user with check(id=10);

explain select * from table_name;
It will apply the mandatory filter in the plan.

to see permissions use
\z table_name

drop policy if exists pol_name on table_name cascade|restrict;
select * from orders;

now rows

postgres=# \d orders
               Table "public.orders"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 custid | integer |           |          |
 billed | boolean |           | not null |
 amount | integer |           |          |
Policies (row security enabled): (none)

postgres=# alter table orders disable row level security
postgres-# ;
ALTER TABLE
postgres=# \d orders
               Table "public.orders"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 custid | integer |           |          |
 billed | boolean |           | not null |
 amount | integer |           |          |

postgres=# \q
-bash-4.2$ psql -Ujoe postgres
psql (16.2)
Type "help" for help.

postgres=> select * from orders limit 10;
 custid | billed | amount
--------+--------+--------
    101 | t      |   1024
    102 | f      |   2048
    103 | t      |   3072
    104 | f      |   4096
    105 | t      |   5120
    106 | f      |   6144
    107 | t      |   7168
    108 | f      |   8192
    109 | t      |   9216
    110 | f      |  10240


can create more then one policy to table.
can alter or rename policy.

