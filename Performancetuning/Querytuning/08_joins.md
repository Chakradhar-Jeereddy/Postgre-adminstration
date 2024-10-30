* Joins Test Case
 - Sample Data:
```
create table emp(deptid int,empid int);
create table  dept (deptid int, salary int);

insert into emp(deptid,empid)
select n,random()*1000
from generate_series(1,50000) n;

insert into dept(deptid,salary)
select n,random()*1000
from generate_series(1,20000) n;

create index idx_dept1 on emp(deptid);
create index idx_dept2 on dept(deptid);
```
- Nested Loop:
 - nested loop: Joins two tables by fetching the result from one table and querying the other table for each row from the first.
 - The least performant form of join.
 - Fast to produce first record.
 - Negative performance possible if the second child is slow.
 - Only join capable of executing CROSS JOIN.
 - Only join capable of inequality join conditions.

- Statement:
```
postgres=# explain analyze select * from emp e, dept d where e.deptid < d.deptid; => non equi join was used.
                                                             QUERY PLAN
 Nested Loop  (cost=0.29..9174383.00 rows=333333333 width=16) (actual time=0.039..54
516.238 rows=199990000 loops=1)
   ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time=0.009
..9.488 rows=20000 loops=1)
   ->  Index Scan using idx_dept1 on emp e  (cost=0.29..292.03 rows=16667 width=8) (
actual time=0.008..1.571 rows=10000 loops=20000)
         Index Cond: (deptid < d.deptid)
 Planning Time: 0.364 ms
 Execution Time: 61879.026 ms

Note - The plan did full scan on outer table dept (2000 rows), for each row it scanned inner table emp for matching rows.
       and it did 20000 loops. Those many round trips/random access or execution.

postgres=# explain analyze select * from emp e, dept d where e.deptid= d.deptid;
 ->  Merge Join  (cost=0.61..1535.86 rows=20000 width=16) (actual time=0.028..21.459 rows=20000 loops=1)
   Merge Cond: (e.deptid = d.deptid)
   ->  Index Scan using idx_dept1 on emp e  (cost=0.29..1531.29 rows=50000 width=8) (actual time=0.006..5.124 rows=20001 loops=1)
   ->  Index Scan using idx_dept2 on dept d  (cost=0.29..620.29 rows=20000 width=8) (actual time=0.005..6.571 rows=20000 loops=1)
 Planning Time: 0.200 ms
 Execution Time: 22.672 ms
(6 rows)

case 1 -
 - drop index on emp table.
postgres=# drop index idx_dept1;
DROP INDEX
postgres=# explain analyze select * from emp e, dept d where e.deptid < d.deptid;
                                                             QUERY PLAN
 Nested Loop  (cost=0.29..9182806.00 rows=333333333 width=16) (actual time=0.017..56
434.171 rows=199990000 loops=1)
   ->  Seq Scan on emp e  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.004.
.16.764 rows=50000 loops=1)
   ->  Index Scan using idx_dept2 on dept d  (cost=0.29..116.97 rows=6667 width=8) (
actual time=0.003..0.650 rows=4000 loops=50000)
         Index Cond: (deptid > e.deptid)
 Planning Time: 0.086 ms
 Execution Time: 64127.139 ms

Note - It uses emp as outer table becuase it doesn't have index, the table which as index it is used as inner table 
       for faster search.

Case 2 -
- drop index on dept table.
   drop index idx_dept2 ;
   explain analyze select * from emp e, dept d where e.deptid < d.deptid;
                                                             QUERY PLAN
 Nested Loop  (cost=0.29..9174383.00 rows=333333333 width=16) (actual time=0.022..55
497.317 rows=199990000 loops=1)
   ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time=0.005
..7.323 rows=20000 loops=1)
   ->  Index Scan using idx_dept1 on emp e  (cost=0.29..292.03 rows=16667 width=8) (
actual time=0.008..1.586 rows=10000 loops=20000)
         Index Cond: (deptid < d.deptid)
 Planning Time: 0.157 ms
 Execution Time: 63170.163 ms

Note - Now emp was used as inter table as it had index and dept went for full scan because of no index.
```
### Hash Joins:
 - hash joins: The hash join loads the candidate records from one side of the join into a hash table which 
   is then probed for each record from the other side of the join.
   In details - It looads records of inner table into hash_table in work_mem and use other table as 
   outer/probe table, for each row in outer table it will probe hash table for matching records.
   Hash join can only be used for equality join conditions and no index on join column.
 - The most performant for joining a large table against a small table and small table being inner table.
 - Slow start due to hashing the smaller table.
 - Performance is negatively impacted if table stats out of date and incorrect.

- Statement:
```
postgres=# drop index idx_dept2;
DROP INDEX
postgres=#
postgres=# drop index idx_dept1;
DROP INDEX
postgres=# explain analyze select * from emp e, dept d where d.deptid = e.deptid;
The first table after from clause is outer table and the second one is inner table
                                                      QUERY PLAN

 Hash Join  (cost=539.00..1648.50 rows=20000 width=16) (actual time=6.892..29.262 ro
ws=20000 loops=1)
   Hash Cond: (e.deptid = d.deptid)
   ->  Seq Scan on emp e  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.010.
.7.304 rows=50000 loops=1)
   ->  Hash  (cost=289.00..289.00 rows=20000 width=8) (actual time=6.829..6.831 rows
=20000 loops=1)
         Buckets: 32768  Batches: 1  Memory Usage: 1038kB
         ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time
=0.006..2.687 rows=20000 loops=1)
 Planning Time: 0.208 ms
 Execution Time: 30.670 ms
(8 rows)

postgres=# show work_mem;
 work_mem
----------
 4MB
(1 row)

postgres=# show hash_mem_multiplier;
 hash_mem_multiplier
---------------------
 2
(1 row)

postgres=# set work_mem='128kB';
SET
postgres=# explain analyze select * from  dept d,emp e where e.deptid = d.deptid;
                                                      QUERY PLAN
 Hash Join  (cost=618.00..2198.50 rows=20000 width=16) (actual time=3.236..20.873 ro
ws=20000 loops=1)
   Hash Cond: (e.deptid = d.deptid)
   ->  Seq Scan on emp e  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.009.
.2.909 rows=50000 loops=1)
   ->  Hash  (cost=289.00..289.00 rows=20000 width=8) (actual time=3.206..3.207 rows
=20000 loops=1)
         Buckets: 8192  Batches: 8  Memory Usage: 164kB    (164KB of work_mem is allocatded not enough to fit whole table)
         ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time
=0.003..1.011 rows=20000 loops=1)
 Planning Time: 0.075 ms
 Execution Time: 21.421 ms
(8 rows)

Note - To reduce the execution time and number of batches, increase the value of work_mem.

postgres=# set work_mem='4MB';
SET
postgres=# explain analyze select * from  dept d,emp e where e.deptid = d.deptid;
                                                      QUERY PLAN

 Hash Join  (cost=539.00..1648.50 rows=20000 width=16) (actual time=2.884..13.696 ro
ws=20000 loops=1)
   Hash Cond: (e.deptid = d.deptid)
   ->  Seq Scan on emp e  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.010.
.3.143 rows=50000 loops=1)
   ->  Hash  (cost=289.00..289.00 rows=20000 width=8) (actual time=2.846..2.847 rows
=20000 loops=1)
         Buckets: 32768  Batches: 1  Memory Usage: 1038kB
         ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time
=0.003..1.135 rows=20000 loops=1)
 Planning Time: 0.078 ms
 Execution Time: 14.256 ms
(8 rows)

Note - There is just 1 batch and less execution time.

```
- merge join: This is very fast for large tables, but input for join should be presorted.
- THe join columns should have indexes to perform index looks for outer and inner table.
 - Can only be used for equality join conditions.
 - Generally, the most performant for large data sets.
 - Requires ordered inputs - which can require slow sorts or index scans.
 - Slow to start up, as all index tuples are read and sorted.
 - All index tuples are first read and sorted.

- Statement: (Create Index)
```
create index idx_dept1 on emp(deptid);
create index idx_dept2 on dept(deptid);
postgres=# explain analyze select * from emp e, dept d where e.deptid = d.deptid;
                                                            QUERY PLAN
 Merge Join  (cost=0.61..1535.86 rows=20000 width=16) (actual time=0.018..10.375 rows=2000
0 loops=1)
   Merge Cond: (e.deptid = d.deptid)
   ->  Index Scan using idx_dept1 on emp e  (cost=0.29..1531.29 rows=50000 width=8) (actua
l time=0.005..3.162 rows=20001 loops=1)
   ->  Index Scan using idx_dept2 on dept d  (cost=0.29..620.29 rows=20000 width=8) (actua
l time=0.009..2.692 rows=20000 loops=1)
 Planning Time: 0.250 ms
 Execution Time: 10.940 ms
(6 rows)

postgres=# explain analyze select * from dept d,emp e where e.deptid = d.deptid;
                                                            QUERY PLAN
 Merge Join  (cost=0.61..1535.86 rows=20000 width=16) (actual time=0.014..9.225 rows=20000
 loops=1)
   Merge Cond: (d.deptid = e.deptid)
   ->  Index Scan using idx_dept2 on dept d  (cost=0.29..620.29 rows=20000 width=8) (actua
l time=0.006..2.360 rows=20000 loops=1)
   ->  Index Scan using idx_dept1 on emp e  (cost=0.29..1531.29 rows=50000 width=8) (actua
l time=0.004..2.455 rows=20001 loops=1)
 Planning Time: 0.135 ms
 Execution Time: 9.775 ms
(6 rows)

set enable_mergejoin=off;
explain analyze select * from dept d,emp e where e.deptid = d.deptid;
                                                      QUERY PLAN
 Hash Join  (cost=539.00..1648.50 rows=20000 width=16) (actual time=3.732..16.277 rows=200
00 loops=1)
   Hash Cond: (e.deptid = d.deptid)
   ->  Seq Scan on emp e  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.013..2.978
 rows=50000 loops=1)
   ->  Hash  (cost=289.00..289.00 rows=20000 width=8) (actual time=3.656..3.657 rows=20000
 loops=1)
         Buckets: 32768  Batches: 1  Memory Usage: 1038kB
         ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time=0.005
..1.334 rows=20000 loops=1)
 Planning Time: 0.141 ms
 Execution Time: 16.923 ms   => expensive then merge
(8 rows)


postgres=# explain analyze select * from dept d,emp e where e.deptid < d.deptid; non equi join so nested loop.
                                                             QUERY PLAN
 Nested Loop  (cost=0.29..9174383.00 rows=333333333 width=16) (actual time=0.059..59231.57
1 rows=199990000 loops=1)
   ->  Seq Scan on dept d  (cost=0.00..289.00 rows=20000 width=8) (actual time=0.035..10.3
05 rows=20000 loops=1)
   ->  Index Scan using idx_dept1 on emp e  (cost=0.29..292.03 rows=16667 width=8) (actual
 time=0.009..1.744 rows=10000 loops=20000)
         Index Cond: (deptid < d.deptid)
 Planning Time: 1.102 ms
 Execution Time: 66978.242 ms   => very expensive.

set enable_hashjoin=off;
set enable hashjoin=on;
set enable_mergejoin=on;
set enable_mergejoin=off;
```

- Aggregation
```
postgres=# explain select count(*) from dept;
                           QUERY PLAN
----------------------------------------------------------------
 Aggregate  (cost=339.00..339.01 rows=1 width=8)
   ->  Seq Scan on dept  (cost=0.00..289.00 rows=20000 width=0)
(2 rows)

```
- limit
```
postgres=# explain select count(*) from dept limit 10;
                              QUERY PLAN
----------------------------------------------------------------------
 Limit  (cost=339.00..339.01 rows=1 width=8)
   ->  Aggregate  (cost=339.00..339.01 rows=1 width=8)
         ->  Seq Scan on dept  (cost=0.00..289.00 rows=20000 width=0)
(3 rows)
```
- sort
```
postgres=# explain select deptid,count(*) from dept group by deptid;
                           QUERY PLAN
----------------------------------------------------------------
 HashAggregate  (cost=389.00..589.00 rows=20000 width=12)
   Group Key: deptid
   ->  Seq Scan on dept  (cost=0.00..289.00 rows=20000 width=4)
(3 rows)
```





