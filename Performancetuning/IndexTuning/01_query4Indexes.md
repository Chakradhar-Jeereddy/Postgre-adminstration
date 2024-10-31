```
 Useful Queries to check Indexes and its Utilization

--List all Indexes in public schema:
SELECT
tablename as "TableName",
indexname as "Index Name",
indexdef as "Index script"
FROM
pg_indexes
WHERE
schemaname = 'public'
ORDER BY
tablename,
indexname;


--List all the indexes in a table and whether it is Primary or Unique key:
select 
    c.relnamespace::regnamespace as schema_name,
    c.relname as table_name,
    i.indexrelid::regclass as index_name,
    i.indisprimary as is_pk,
    i.indisunique as is_unique
from pg_index i
join pg_class c on c.oid = i.indrelid
where c.relname = 'pgbench_tellers';

-- Unused Indexes:
select * from pg_stat_all_indexes where idx_scan = 0 and schemaname='public';
or
SELECT
relname AS table_name,
indexrelname AS index_name,
pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
idx_scan AS index_scan_count
FROM
pg_stat_user_indexes
WHERE
idx_scan < 100
ORDER BY
index_scan_count ASC,
pg_relation_size(indexrelid) DESC;


-- Does table needs an Index:
SELECT relname, seq_scan-idx_scan AS too_much_seq, CASE WHEN seq_scan-idx_scan>0 
THEN 'Missing/Ineff Index' 
ELSE 'OK' END, 
pg_relation_size(relname::regclass) AS rel_size, seq_scan, idx_scan FROM pg_stat_all_tables WHERE schemaname='public' AND pg_relation_size(relname::regclass)>80000 ORDER BY too_much_seq DESC;
	

-- How many indexes are in cache:
SELECT sum(idx_blks_read) as idx_read, sum(idx_blks_hit) as idx_hit FROM pg_statio_user_indexes;


-- Index % usage:
SELECT relname, 100 * idx_scan / (seq_scan + idx_scan) percent_of_times_index_used, n_live_tup rows_in_table FROM pg_stat_user_tables WHERE (seq_scan + idx_scan) > 0 ORDER BY n_live_tup DESC;


-- Duplicate Indexes:
SELECT ni.nspname || '.' || ct.relname AS "table", 
       ci.relname AS "dup index",
       pg_get_indexdef(i.indexrelid) AS "dup index definition", 
       i.indkey AS "dup index attributes",
       cii.relname AS "encompassing index", 
       pg_get_indexdef(ii.indexrelid) AS "encompassing index definition",
       ii.indkey AS "enc index attributes"
  FROM pg_index i
  JOIN pg_class ct ON i.indrelid=ct.oid
  JOIN pg_class ci ON i.indexrelid=ci.oid
  JOIN pg_namespace ni ON ci.relnamespace=ni.oid
  JOIN pg_index ii ON ii.indrelid=i.indrelid AND
                      ii.indexrelid != i.indexrelid AND
                      (array_to_string(ii.indkey, ' ') || ' ') like (array_to_string(i.indkey, ' ') || ' %') AND
                      (array_to_string(ii.indcollation, ' ')  || ' ') like (array_to_string(i.indcollation, ' ') || ' %') AND
                      (array_to_string(ii.indclass, ' ')  || ' ') like (array_to_string(i.indclass, ' ') || ' %') AND
                      (array_to_string(ii.indoption, ' ')  || ' ') like (array_to_string(i.indoption, ' ') || ' %') AND
                      NOT (ii.indkey::integer[] @> ARRAY[0]) AND -- Remove if you want expression indexes (you probably don't)
                      NOT (i.indkey::integer[] @> ARRAY[0]) AND -- Remove if you want expression indexes (you probably don't)
                      i.indpred IS NULL AND -- Remove if you want indexes with predicates
                      ii.indpred IS NULL AND -- Remove if you want indexes with predicates
                      CASE WHEN i.indisunique THEN ii.indisunique AND
                         array_to_string(ii.indkey, ' ') = array_to_string(i.indkey, ' ') ELSE true END
  JOIN pg_class ctii ON ii.indrelid=ctii.oid
  JOIN pg_class cii ON ii.indexrelid=cii.oid
 WHERE ct.relname NOT LIKE 'pg_%' AND
       NOT i.indisprimary
 ORDER BY 1, 2, 3;

Useful Functions to Find size of Objects:
pg_size_pretty() function to format the size.
pg_relation_size() function to get the size of a table.
pg_total_relation_size() function to get the total size of a table.
pg_database_size() function to get the size of a database.
pg_indexes_size() function to get the size of an index.
pg_total_index_size() function to get the size of all indexes on a table.
pg_tablespace_size() function to get the size of a tablespace.
pg_column_size() function to obtain the size of a column of a specific type.

```
