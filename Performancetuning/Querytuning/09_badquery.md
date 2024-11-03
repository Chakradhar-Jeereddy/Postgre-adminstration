-- Query Tuning 
 - Example 1: (Joins Instead of IN).

```
Explain analyze select * from emp where deptid IN (SELECT deptid from dept where salary>800);
    - How many results populated out of inner query.More the results the more the query will slow down.
      GO for joins if inner query returns many results.

      Query plan
    Hash Semi Join  (cost=386.44..1281.91 rows=3795 width=8) (actual time=1.551..7.948
rows=3787 loops=1)
   Hash Cond: (emp.deptid = dept.deptid)
   ->  Seq Scan on emp  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.005..2
.612 rows=50000 loops=1)
   ->  Hash  (cost=339.00..339.00 rows=3795 width=4) (actual time=1.539..1.539 rows=
3787 loops=1)
         Buckets: 4096  Batches: 1  Memory Usage: 166kB
         ->  Seq Scan on dept  (cost=0.00..339.00 rows=3795 width=4) (actual time=0.
005..1.188 rows=3787 loops=1)
               Filter: (salary > 800)
               Rows Removed by Filter: 16213   => This can be optimized using index.
 Planning Time: 0.163 ms
 Execution Time: 8.063 ms
(10 rows)
```
-  Query Rewrite:
  ```
    explain analyze select emp.* from emp JOIN dept on emp.deptid=dept.deptid where dept.salary>800;
     Query plan
 Merge Join  (cost=564.90..1287.11 rows=3795 width=8) (actual time=1.742..6.032 rows
=3787 loops=1)
   Merge Cond: (emp.deptid = dept.deptid)
   ->  Index Scan using idx_dept1 on emp  (cost=0.29..1531.29 rows=50000 width=8) (a
ctual time=0.007..2.273 rows=20000 loops=1)              => It used index and scanned only 20000 rows instead of 5000.
   ->  Sort  (cost=564.61..574.10 rows=3795 width=4) (actual time=1.730..1.887 rows=
3787 loops=1)
         Sort Key: dept.deptid
         Sort Method: quicksort  Memory: 97kB    => to use merge join rows should be sorted
         ->  Seq Scan on dept  (cost=0.00..339.00 rows=3795 width=4) (actual time=0.
008..1.544 rows=3787 loops=1)   => 3787 rows met the criteria
               Filter: (salary > 800)
               Rows Removed by Filter: 16213  => This can be eliminated by using index.
 Planning Time: 0.225 ms
 Execution Time: 6.161 ms
```
 - Example 2: (Aggregate and Join)
```
CREATE TABLE bill(id int, status varchar);
INSERT INTO bill  VALUES (1, 'billed'), (2, 'non-billed');
CREATE TABLE order_item (orderid serial,order_status int);
INSERT INTO order_item (order_status)
SELECT  x % 2 + 1
FROM    generate_series(1, 1000000) AS x;

Explain analyze  SELECT status,count(*)
               FROM    bill AS a, order_item AS b
              WHERE   a.id = b.order_status
              GROUP BY 1;
```
-- Rewrite:
```
explain analyze WITH x AS
( 
  SELECT order_status,count(*) AS res
  FROM   order_item AS a
  GROUP BY 1
)
SELECT   status,res
FROM     x, bill AS y
WHERE    x.order_status = y.id;
```

 - Example 3: (Select specific column instead of all columns)
    Basically selecting only the required columns will reduce the query execution time.
    Explain analyze select * from pg_Stats;
    Query Rewrite:
    Explain analyze select schemaname,tablename,attname from pg_stats;

 - Example 4: (Create index on Orderby clause to avoid sorting everytime)
```
    Explain analyze select * from emp order by empid;
        Quey plan (without index on column used under order by clause need to be sorted each time)
    -> Sort  (cost=4624.41..4749.41 rows=50000 width=8) (actual time=9.988..13.402 rows=50
000 loops=1)
   Sort Key: empid
   Sort Method: quicksort  Memory: 3099kB
   ->  Seq Scan on emp  (cost=0.00..722.00 rows=50000 width=8) (actual time=0.008..3
.291 rows=50000 loops=1)
 Planning Time: 0.112 ms
 Execution Time: 14.743 ms
(6 rows)

Create index idx_emp1 on emp(empid);
Explain analyze select * from emp order by empid;
 Index Scan using idx_emp1 on emp  (cost=0.29..1822.25 rows=50000 width=8) (actual t
ime=0.014..10.949 rows=50000 loops=1)
 Planning Time: 0.132 ms
 Execution Time: 12.362 ms
(3 rows)
```
 - Example 5: (Wildcards at the end)
```
explain analyze select firstname from staff where firstname like '%ab%';
    Seq Scan on staff  (cost=0.00..41.94 rows=20 width=7) (actual time=0.008..0.122 row
s=47 loops=1)
   Filter: ((firstname)::text ~~ '%ab%'::text)
   Rows Removed by Filter: 1948
 Planning Time: 0.076 ms
 Execution Time: 0.133 ms
(5 rows)
```

Query Rewrite:
explain analyze select firstname from staff where firstname like 'ab%';
 Seq Scan on staff  (cost=0.00..41.94 rows=1 width=7) (actual time=0.010..0.202 rows
=14 loops=1)  => returned only 14 rows
   Filter: ((firstname)::text ~~ 'ab%'::text)
   Rows Removed by Filter: 1981
 Planning Time: 0.063 ms
 Execution Time: 0.215 ms
(5 rows)
```


