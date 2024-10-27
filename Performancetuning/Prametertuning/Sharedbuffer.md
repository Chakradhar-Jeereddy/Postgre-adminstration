```
-- Parameters that affect or enhance the system performance
shared_buffers         work_mem
maintenance_work_mem   AutoVacumm_work_mem
```

```
Shared_buffer determines how much memory is dedicated to cluster for caching data.
Unlike other RDMS systems, PG doesn't do direct I/O to the disk.
It uses its own buffer and a kernal buffer called OS cache in order to do its caching.

Scenarios
Assume a user process is requesting some data from the database, it first look for the data in the shared buffer.
If found, it will be send back to the user process.
If not a request is sent to the OS cache, it also holds some data, if not available then the request goes to the disk.
Disk is not involved when data is found in shared buffer or OS cache.


How read works?
It is bottom to top, from disk to buffer.

How write works?
It is top to bottom, from buffer to disk.

Postgresql is using two memory components for caching data.
1) Shared buffer is part postgresql architecture and it is a internal component and maintained by PGSQL.
2) OS cache is outside PGSQL and maintained by OS kernel.
3) In other RDBMS system we tend to allocate more memory to the database so it performs faster.
4) However, in postgresql it is little bit different, there is another component outside the database.
5) So, we don't allocate more memory to the shared buffer, becuase the OS cache will not have more memory, it will be undersized.
6) Just becuase it is undersized, PGSQL can't by pass the OS cache and go directly to the disk, it has to go through OS cache.
7) If the middle layer is under sized, the data going from disk to buffer(reads) and going from buffer to disk(writes) will suffer.
8) Assume you're inserting large amount of data, the shared buffer flush the data in OS cache.
9) The OS cache will not write all data rightaway to the disk, becuase it doesn't want to cause I/O contention
   by writing huge amount of data at once. It will spread the write operation accross a span of time.
   So the disk doesn't get affected. This is the important function of OS cache.

What is double caching?
Assume I rebooted the clusted and the cluster started.
OS cache and Shared buffer is empty, a user requested data, the data is loaded to OS cache from there to shared buffer.
Same data is there in OS cache and shared buffer, data is duplicated, we are wasting OS cache by having duplicate pages.
Not really, the algorithum for shared buffer and OS cache is different.
OS cache works on LRU(Least recently used) and shared buffer works on clock sweep algorithm(Controls buffer allocation and eviction)
Chances of duplicate pages are minimal.

Caution -
The value of shared_buffers should never be set to reserve all of the system RAM for Postgresql.
The default value of shared buffer is 128MB not high enough for production system.
Recomended value of 25%-40% of RAM is optimal per PSQL documentation for system stability, start with 25%.
As the system mature and based on our observation, we can fine tune its size.
Offical document says that anything above 40% of system memory for shared_buffers is not going to yeild better results.

```







