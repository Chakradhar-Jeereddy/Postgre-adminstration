* The amount of memory to be used by internal sort operations and hash tables before writing to temporary files on disk
* Sort operations are used by order by, distinct,merge join operations. Hash tables are used in hash joins and has-based aggregation.
* Complex query, several sort or hash operations might be running in parallel, and each operation will generally
  be allowed to use as much memory as this value.
* The default value is four megabytes(4MB).
* The memory limit for a hash table is computed by multiplying work_mem by hash_mem_multiplier.

