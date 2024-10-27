* When a row deleted or updated in table, it creates a dead tuple.
* For update, a duplicate entry of the record with new value is stored.
* The old row with old value is marked as dead row.
* Similarly when we delete a record, the data is deleted but space the is not released

-- Issues with dead tuple
  * Bloat - Physical size of table on disk is larget then its actual data size. Also called fragmentation.

-- Issues with bloating
   * Slower queries and Increase I/O.
   * Slower lookup times due to index bloating
   * Wasted disk space
   * Replication lag

-- How to address this issue
   * Remove bloat
   * Enable auto vacuum
   * Reclaim storage by removing obsolete/dead tuples
   * Analyze tables
   * Butter execution plan.

-- How does Auto vacuum works
  * Stage 1 - It scans the table for dead tuples, picks the transaction id of that dead tuple and places them in
  * Miantenance work mem/Autovacuum work mem (transaction IDs of dead tuple are stored.
  * Stage 2 - It checks the indexes of that perticular table with transaction id and delete those dead rows.

-- How to tune the Autovacuum?
   * Table has 200mb of dead tuple, the autovacuum_mem is 64MB, the worker takes 64MB of transaction ids
   * stores in main_work_mem and find index dead tuples and removes it from table and it goes for next 64mb.
     So to process 200MB of dead tuples it take 3 cycles. Each cycle is expansive, it needs resources.

-- Tuning opportunity is reduce the number of cycle by increasing workmem.
   * How is doing the vacuuming, there is master process called launcher, we calls a worker process to do the works
   * Autovacuum_max_workets default is 3 workers.
   * Autovacuum_naptime = default 1 minute, every one minute it scans for dead rows

Scenario 1:

5 databases in cluster with updates and deletes happening
Autovacuum spwan 1 worker assing to one database(which database it choose, not in our control)
After nap time of 1 minute, worker 2 is assigned to another database
after nap time, workder 3 is assigned to another database.
The remaining 2 databases need to wait for one the three workers to finish vacuuming.
Problem is 3 workers but 5 databases.
Solution - Increase number of workers (need more resources)
           Reduce nap time, works more aggresively
           Increase work mem
Scenario2:
1 large database with many tables being updated and deleted.
worker1 for 1 table
worker2 for 2 table
worker3 for 3 table

After each nap time it is checking the amount of dead rows, there is threshold only when it is reached, the 
vacuum is performed.

```
postgres=# show autovacuum_max_workers;
-[ RECORD 1 ]----------+--
autovacuum_max_workers | 3

postgres=#  show autovacuum_nap_time;
ERROR:  unrecognized configuration parameter "autovacuum_nap_time"
postgres=#  show autovacuum_naptime;
-[ RECORD 1 ]------+-----
autovacuum_naptime | 1min

postgres=# show Autovacuum_vacuum_cost_limit;
-[ RECORD 1 ]----------------+---
autovacuum_vacuum_cost_limit | -1

postgres=# show vacuum_cost_limit;
-[ RECORD 1 ]-----+----
vacuum_cost_limit | 200

postgres=# show Autovacuum_vacuum_cost_delay;
-[ RECORD 1 ]----------------+----
autovacuum_vacuum_cost_delay | 2ms

postgres=#  show Autovacuum_vacuum_cost_page_hit
postgres-# ;
ERROR:  unrecognized configuration parameter "autovacuum_vacuum_cost_page_hit"
postgres=# show vacuum_cost_page_hit;
-[ RECORD 1 ]--------+--
vacuum_cost_page_hit | 1

postgres=# show vacuum_cost_page_miss;
-[ RECORD 1 ]---------+--
vacuum_cost_page_miss | 2

postgres=# show vacuum_cost_page_dirty;
-[ RECORD 1 ]----------+---
vacuum_cost_page_dirty | 20
```

## Suggestion - Either increase the workers or reduce the nap time, don't do both at same time.
                Its a game of permitation and combination and extensive monitoring.


* Autovacuum_vacuum_cost_limit - Amount of work autovacuum does in one cycle
* Controles the amount of CPU and I/O resources that a worker can consume
* If the value is too high, it can slow down other queries
* If it is set to low, the vacuum proces may not reclaim space which causes table to become larger over time.

* Autovacuum_vacuum_cost_delay - 2 milisecons(Default) Number of mille seconds it should asleep after it has reached the cost limit.
* If autovacuum is performing very aggresively then increase this value(breaks).
* To be more aggressive reduce the break time.

show vacuum_cost_page_hit=1; (found in shared buffer)
show vacuum_cost_page_miss=2; (not in shared buffer)
show vacuum_cost_page_dirty=20; (page is dirty) found in shared buffer and are dirty

1000ms = 1s
1000/2 = 500 (breaks since delay is 2ms)

cost limit is 200
(200/1)*500*8kb= 782mb/sec
200/buffer*breaks*pagesize

(200/2)*500*8kb= 390mb/sec (read from disk)

(200/10)*500*8kb= 39mb/sec

We can manipulate this value
To do more work increase clost limit to 1000, when the limit is increases, you are taxing the system resources
(1000/1)*500*8kb= 3.9g/sec

Two approches
1) Aggressive - Want the process to run more offen, less breaks
                Reduce the cost delay and increase cost limit
2) Consurvatie - Want more work to be done with more break time
                 Incease the cost limit and cost delay

3) Be aggrssive during off peak hours and conservative during lass peak hours.
   Using script and schedule it.
   alter system autovacuum_vacuum_cost_limit='2000'
   select pg_reload_conf();









     
      
