-- Query Tuning 
 - Example 1: (Joins Instead of IN).

 - Explain analyze select * from emp where deptid IN (SELECT deptid from dept where salary>800);
-  Query Rewrite:
    explain analyze select emp.* from emp JOIN dept on emp.deptid=dept.deptid where dept.salary>800;
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
    Explain analyze select * from pg_Stats;
    Query Rewrite:
    Explain analyze select schemaname,tablename,attname from pg_stats;

 - Example 4: (Create index on Orderby clause to avoid sorting everytime)
    Explain analyze select * from emp order by empid;
    Create index idx_emp1 on emp(empid);
    Explain analyze select * from emp order by empid; 

 - Example 5: (Wildcards at the end)
    SELECT City FROM Customers WHERE City LIKE ‘%Char%’
 - Results:
    Charleston, Charleston, Charlton, Cape Charles, Crab Orchard, and Richardson.

Query Rewrite:
SELECT City FROM Customers WHERE City LIKE ‘Char%’
Results:
Charleston, Charlotte, and Charlton.


