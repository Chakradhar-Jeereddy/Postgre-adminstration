```
-- Data,wall_files,temp_files,archives and binaries at same location cause single point of filure.
-- Disk I/O contention, disk unable to keep up with the amount of requests and transactions being generated.
-- Keep binaries on spinning disks
-- Data on high speed or low speed if size is huge.
-- Wal files should be kept on high speed SSD disk
-- Temp disk can be high or slow spin disk
-- Archives should be kept on separate location, important to perform point intime recover.
```
