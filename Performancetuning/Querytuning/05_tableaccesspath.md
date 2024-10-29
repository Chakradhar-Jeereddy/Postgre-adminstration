## Sequential scan
- The seq scan operation scans the entire replation (table) in a sequenctial (serial) order.
- We can apply filters while reading data, but all the data has to be read first and then filtered.
- Sequential scans are typically slow and should be avoid in production environment for large tables.
- Small tables are good candidates for Sequenctial scans.

-- Example
```
postgres=# explain analyze select * from pgbench_tellers;
                                                  QUERY PLAN
 Seq Scan on pgbench_tellers  (cost=0.00..8.00 rows=500 width=352) (actual time=0.01
6..0.057 rows=500 loops=1)   => all rows scanned and fetched
 Planning Time: 0.293 ms
 Execution Time: 0.096 ms
(3 rows)

postgres=# explain analyze select * from pgbench_tellers where bid=30; (bid column as no index on it)
                                                 QUERY PLAN
 Seq Scan on pgbench_tellers  (cost=0.00..9.25 rows=10 width=352) (actual time=0.019
..0.027 rows=10 loops=1)
   Filter: (bid = 30)
   Rows Removed by Filter: 490  => All rows scanned and then removed 490 rows and fetched 10 rows, query is not efficent
 Planning Time: 0.045 ms
 Execution Time: 0.037 ms
(5 rows)
```

# Index scans better when looking for specific data or small set of data: 
 - The index scan walks through leaf nodes to find all the matching entries or rowids and fetches the corresponding 
   - table data. It is an index range scan followed by a table access by index rowid operation.
- Index scan characteristics
  - Random access is much slower than sequential I/O.
  - Requires additional I/O to access index.
  - Porentially reads the same block multiple times.
  - Produces ordered output.

```
postgres=# explain analyze select * from pgbench_tellers where tid=204;
 -> It will check the index(which has two values(index key and pointer to table row)) 
     and finds the location and pick the row from table
Index Scan using pgbench_tellers_pkey on pgbench_tellers  (cost=0.27..8.29 rows=1 w
idth=352) (actual time=0.052..0.054 rows=1 loops=1)
   Index Cond: (tid = 204)
 Planning Time: 0.061 ms
 Execution Time: 0.068 ms
```

- Bitmap index/heap scans: A plain index scan fetches one tuple-pointer at a time from the index and immediately visits
                           that tuple in the table. A bitmap scan fetches all the tuple-pointers from the index in one go,
                           sorts them using an in-memory "bitmap" data structure and then visits the table tuples in physcial
                           tuple lication order.
  - Characteristics of a bitmap index/heap scan:
        - Sequential I/O with index selectivity.
        - Slow to startup as all index rows are read and sorted.
        - Often selected for in and any array operators, as well as low selectivity index scans.
        - Can combine multiple indexes.
        - Produces unordered output.
```
postgres=#  explain analyze select * from pgbench_tellers where tid>0 and tid<100;
                                                          QUERY PLAN
 Bitmap Heap Scan on pgbench_tellers  (cost=5.29..9.77 rows=99 width=352) (actual ti
me=0.326..0.330 rows=99 loops=1)
   Recheck Cond: ((tid > 0) AND (tid < 100))
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on pgbench_tellers_pkey  (cost=0.00..5.26 rows=99 width=0)
(actual time=0.317..0.317 rows=99 loops=1)
         Index Cond: ((tid > 0) AND (tid < 100))
 Planning Time: 1.400 ms
 Execution Time: 0.358 ms
(7 rows)

postgres=# explain analyze select * from pgbench_tellers where tid>0 and tid<400;
                                                   QUERY PLAN
- Ful scan used because it is larger data set more then 70% of table data fetched.
 Seq Scan on pgbench_tellers  (cost=0.00..10.50 rows=399 width=352) (actual time=0.0
10..0.086 rows=399 loops=1)
   Filter: ((tid > 0) AND (tid < 400))
   Rows Removed by Filter: 101
 Planning Time: 0.117 ms
 Execution Time: 0.117 ms
(5 rows)

 explain analyze select * from pgbench_tellers where tid=40;
                                                               QUERY PLAN
 Index Scan using pgbench_tellers_pkey on pgbench_tellers  (cost=0.27..8.29 rows=1 w
idth=352) (actual time=0.017..0.018 rows=1 loops=1)
   Index Cond: (tid = 40)
 Planning Time: 0.050 ms
 Execution Time: 0.029 ms

```
- Index only scan:
    There is no table access needed becuase the index has all columns to statisfy the query.
```
 explain analyze select tid from pgbench_tellers where tid<10;
                                                                QUERY PLAN
 Index Only Scan using pgbench_tellers_pkey on pgbench_tellers  (cost=0.27..4.43 row
s=9 width=4) (actual time=0.016..0.018 rows=9 loops=1)
   Index Cond: (tid < 10)
   Heap Fetches: 0  => this will increase if pages gets modified during the execution(like someone deleted 100 rows)
 Planning Time: 0.059 ms
 Execution Time: 0.032 ms
```



