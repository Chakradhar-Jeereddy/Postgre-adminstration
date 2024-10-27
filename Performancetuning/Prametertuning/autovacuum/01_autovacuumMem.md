* Autovacuum_work_mem specified the amount of memory to be used by each autovacuum worker prcoess.
* Default is -1 (disabled)
* The autovacumm worker can use the memory update 1G max to prevent memory wasting.
* Setting the value higher then 1G has no effect on number of dead tuples that the process can collect while scanning the table.
* When Autovacuum_work_mem is not configured, maintenance workmem will be used.

```
postgres=# show autovacuum_work_mem;
 autovacuum_work_mem
---------------------
 -1
(1 row)

postgres=# show maintenance_work_mem;
 maintenance_work_mem
----------------------
 64MB
(1 row)
```
   
