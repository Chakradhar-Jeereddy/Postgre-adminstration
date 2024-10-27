* load data using pgbench for demo

-- pgbench -i -s 50 postgres

* Test cases
* Work Mem :  4MB Default

* select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
* explain analyze  select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
```
Query plan (for worker0,1, the sort method happening on the disk)
----------
Limit  (cost=379424.97..379434.38 rows=10 width=8) (actual time=1873.446..1885.407
rows=10 loops=1)
   ->  Nested Loop  (cost=379424.97..472661106225.36 rows=502048327326 width=8) (act
ual time=1873.444..1885.403 rows=10 loops=1)
         Join Filter: (a.bid = b.bid)
         ->  Gather Merge  (cost=379424.97..961757.36 rows=5000000 width=8) (actual
time=1873.405..1885.355 rows=1 loops=1)
               Workers Planned: 2
               Workers Launched: 2
               ->  Sort  (cost=378424.94..383633.28 rows=2083333 width=8) (actual ti
me=1820.661..1820.795 rows=1365 loops=3)
                     Sort Key: a.bid
                     Sort Method: external merge  Disk: 31400kB
                     Worker 0:  Sort Method: external merge  Disk: 28976kB
                     Worker 1:  Sort Method: external merge  Disk: 27816kB
                     ->  Parallel Seq Scan on pgbench_accounts a  (cost=0.00..102801
.33 rows=2083333 width=8) (actual time=0.019..738.435 rows=1666667 loops=3)
         ->  Materialize  (cost=0.00..176500.00 rows=5000000 width=4) (actual time=0
.031..0.036 rows=10 loops=1)
               ->  Seq Scan on pgbench_accounts b  (cost=0.00..131968.00 rows=500000
0 width=4) (actual time=0.026..0.028 rows=10 loops=1)
 Planning Time: 0.108 ms
 Execution Time: 1890.254 ms
(16 rows)
```
* SET work_mem = '64MB';
* explain analyze select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
```
Query plan
------------
 Limit  (cost=322451.97..322461.38 rows=10 width=8) (actual time=1651.484..1662.492
rows=10 loops=1)
   ->  Nested Loop  (cost=322451.97..472661049252.36 rows=502048327326 width=8) (act
ual time=1651.482..1662.489 rows=10 loops=1)
         Join Filter: (a.bid = b.bid)
         ->  Gather Merge  (cost=322451.97..904784.36 rows=5000000 width=8) (actual
time=1651.445..1662.447 rows=1 loops=1)
               Workers Planned: 2
               Workers Launched: 2
               ->  Sort  (cost=321451.94..326660.28 rows=2083333 width=8) (actual ti
me=1616.428..1616.550 rows=1365 loops=3)
                     Sort Key: a.bid
                     Sort Method: external merge  Disk: 33328kB
                     Worker 0:  Sort Method: external merge  Disk: 25688kB
                     Worker 1:  Sort Method: external merge  Disk: 29072kB
                     ->  Parallel Seq Scan on pgbench_accounts a  (cost=0.00..102801
.33 rows=2083333 width=8) (actual time=0.038..949.743 rows=1666667 loops=3)
         ->  Materialize  (cost=0.00..176500.00 rows=5000000 width=4) (actual time=0
.031..0.033 rows=10 loops=1)
               ->  Seq Scan on pgbench_accounts b  (cost=0.00..131968.00 rows=500000
0 width=4) (actual time=0.026..0.027 rows=10 loops=1)
 Planning Time: 0.129 ms
 Execution Time: 1665.384 ms
```
* Still sorting on disk but there is reduction in execution time.

* SET work_mem = '1GB';
* explain analyze select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
```
Query plan
Limit  (cost=322451.97..322459.44 rows=10 width=8) (actual time=1513.755..1523.322
rows=10 loops=1)
   ->  Nested Loop  (cost=322451.97..375001049252.36 rows=502048327326 width=8) (act
ual time=1513.753..1523.317 rows=10 loops=1)
         Join Filter: (a.bid = b.bid)
         ->  Gather Merge  (cost=322451.97..904784.36 rows=5000000 width=8) (actual
time=1513.719..1523.273 rows=1 loops=1)
               Workers Planned: 2
               Workers Launched: 2
               ->  Sort  (cost=321451.94..326660.28 rows=2083333 width=8) (actual ti
me=1484.695..1484.807 rows=1365 loops=3)
                     Sort Key: a.bid
                     Sort Method: quicksort  Memory: 104348kB
                     Worker 0:  Sort Method: quicksort  Memory: 99043kB
                     Worker 1:  Sort Method: quicksort  Memory: 100316kB
                     ->  Parallel Seq Scan on pgbench_accounts a  (cost=0.00..102801
.33 rows=2083333 width=8) (actual time=0.030..884.877 rows=1666667 loops=3)
         ->  Materialize  (cost=0.00..156968.00 rows=5000000 width=4) (actual time=0
.026..0.032 rows=10 loops=1)
               ->  Seq Scan on pgbench_accounts b  (cost=0.00..131968.00 rows=500000
0 width=4) (actual time=0.020..0.022 rows=10 loops=1)
 Planning Time: 0.142 ms
 Execution Time: 1524.698 ms
(16 rows)
```
 * set work_mem='12G';
 *  explain analyze  select a.aid from pgbench_accounts a, pgbench_accounts b where a.bid=b.bid order by a.bid limit 10;
```
Now the sorting was done in memory
Limit  (cost=322451.97..322459.44 rows=10 width=8) (actual time=1271.346..1275.975
rows=10 loops=1)
   ->  Nested Loop  (cost=322451.97..375001049252.36 rows=502048327326 width=8) (act
ual time=1271.344..1275.971 rows=10 loops=1)
         Join Filter: (a.bid = b.bid)
         ->  Gather Merge  (cost=322451.97..904784.36 rows=5000000 width=8) (actual
time=1271.277..1275.896 rows=1 loops=1)
               Workers Planned: 2
               Workers Launched: 2
               ->  Sort  (cost=321451.94..326660.28 rows=2083333 width=8) (actual ti
me=1247.084..1247.132 rows=1365 loops=3)
                     Sort Key: a.bid
                     Sort Method: quicksort  Memory: 101203kB
                     Worker 0:  Sort Method: quicksort  Memory: 100892kB
                     Worker 1:  Sort Method: quicksort  Memory: 101613kB
                     ->  Parallel Seq Scan on pgbench_accounts a  (cost=0.00..102801
.33 rows=2083333 width=8) (actual time=0.021..913.855 rows=1666667 loops=3)
         ->  Materialize  (cost=0.00..156968.00 rows=5000000 width=4) (actual time=0
.013..0.019 rows=10 loops=1)
               ->  Seq Scan on pgbench_accounts b  (cost=0.00..131968.00 rows=500000
0 width=4) (actual time=0.009..0.012 rows=10 loops=1)
 Planning Time: 0.135 ms
 Execution Time: 1276.655 ms
````
* RESET work_mem;
* The standard way to find best value for work_mem by is using below formula
* 25% of the total system memory/ max_connections.
* Other way is enable log_temp_files parameter and restart cluster.
* The queries sorting in disk gets logged in alert log
* Test those queries to find the optimal value.
* If its one or two queries ask user to set the value at sessesion level.
* Or set it as userlevel "ALTER ROLE HR SET work_mem TO '1GB'"
* It's tough to get the right value for work_mem perfect, performance tuning is a process not a onetime task.
*   
````
Use this for demo: 
explain analyze select * from pgbench_history order  by aid;
set work_mem='10MB';
SET log_temp_files TO '4MB';
SET trace_sort TO 'on';  (To include resource information).
````



