- Multiversion concurrency control, used to maintain consistency, reads and writes do not block each other.
- In Postgres both before and after image information is stored in same table, unlike mysql and oracle where data is stored in seprate location.
- When a table is created in PostgreSQL, a new record with the table name and the schema name is inserted into the system tables – pg_class and pg_namespace.
- OID - object identifier 
- To find oid of a table use - pg_class
- tableoid is hidden coulmn in each table which containts the oid of the table also stored in pg_class
- Find OID and schema name of the table
```
SELECT pc.oid, pn.nspname, pc.relname
 FROM pg_class pc
 JOIN pg_namespace pn ON pc.relnamespace = pn.oid
 WHERE pn.nspname = 'foo'
 AND pc.relname = 'bar';
 oid | nspname | relname
-------+---------+---------
 31239 | foo | bar
 ```
- Find hidden columns of a table which stores before image
```
postgres=# SELECT attname, format_type (atttypid,atttypmod) FROM pg_attribute
postgres-# WHERE attrelid = 'pgbench_tellers'::regclass::oid;
 attname  |  format_type
----------+---------------
 tableoid | oid
 cmax     | cid
 xmax     | xid
 cmin     | cid
 xmin     | xid
 ctid     | tid
 tid      | integer
 bid      | integer
 tbalance | integer
 filler   | character(84)
```
- Find the transaction id
```
select txid_current();
```
- Unique identifier assigned to each transtaction, it remains same within the begin and end block;
- A transaction ID in PostgreSQL is a 32-bit integer. It is cyclic, which means that it
  starts from 0 and goes up to 4.2 billion (4,294,967,295) and then starts from 0 again.
- The function txid_current() shows the ID of the current transaction
- xmin - is the transaction id that inserted that row, if the rows are inserted as part of same transacton, its xmin value will remain same.
  Any sql statement that started before a transaction lets say 1095, will not abe able to see the future transactions
- xmax - if value of xmax is 0 means the record was never deleted, if it shows a txid that is the transaction which issued a delete

- Two senarios here 
  1) delete issued and not commited.
  2) delete issued and commited.
- If its commited, the values anyway will not be visible in xmax, it will be 0.
- If tran is issued but not deleted, then xmax will store that value and selects will only display the values
```
select * from foo.bar where id=2 and (xmin<=txid_current() and xmax=0 or txid_current() <xmax);
ctid - (page/block number,index within the page)
```
- As all versions of rows are maintained within table due to deletes and updates,
  over a period of time there may be many such deleted tuples still stored in pages
  called dead tuples, the tuples that were inserted and never modified are called live tuples.
- Dead tuples occupy more space and may decrease the performance of the database. The answer to this question is vaccum.
- The autovaccum process takes care of vacumm and analyze tasks on tables 
- Vaccum cleans up dead tuples so that the space can be resused by future inserts and update does a deletion and insertion.
- Analyze collects statistics of table.






