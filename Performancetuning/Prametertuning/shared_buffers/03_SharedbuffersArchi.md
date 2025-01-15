```
Internals of shared buffer

1) Shared buffer is an array of 8KB blocks.
2) Each buffer consist of space to hold one data block and its header
3) The header contains the location of the page in the buffer, the file number and object id.
4) Hash table maintances the buffer ids,pages(meta),buffer tags. It is like parking admin.

````

```
What is buffer allocation?
For the read request, the block from disk is moved to SB, there a buffer is allocated to store the page.
User process will pin the page and reads it, the usage count is now 1.
After reading the user unpin the page and disconnects and page remains in buffer.
Another user comes request same page from hash table, the hash table shares the location, the page gets pinned.
the usage becomes 2 and so on. Maximum it will go is 5.
After it reaches 5 it becomes a hot block(requested often)

At a stage where there is no empty page in shared buffer, based on what criteria the pages are evicted?
The clock sweep algorithem will go through all buffers and it will deduct one count from usage count.
5-1,4-1,3-1,2-1,1-1. Those which are zero are candidates for eviction.

The whole idea is we keep the hot blocks.
Dirty blocks will not have usage count, those will not be removed until writen to the disk.
After checkpoint, those writen to disk and will be removed from buffers.

When shared buffers are under sized, the clock sweep algorithm runs aggresively and evicts the pages and reduce the
usage count of hot blocks.
```

```
Clock sweep process is part of the background writer and buffer managment system.
It determines how frequently dirty pages (modified data) in shared buffers are written to disk.

The interval at which clock sweep runs is controlled by following parameters
bgwriter_delay=200ms  : Determines how often background writer wakes up to scan buffers and perform clock sweep. Default is 200 milli seconds.

bgwriter_lru_maxpages - Number of buffers writen disk each clock sweep pass.
Default: 100 pages
bgwriter_lru_multiplier - Adjusts the aggresiveness of bgwriter by calculating how many buffers to target based on future needs.
Default is 2 (aggressive)

During clock sweep scan of shared buffer it identified dirty buffers not writen to disk based on above
settings. Ensures enough clean buffers are available for incomming connections.
```


