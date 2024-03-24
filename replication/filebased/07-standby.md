## Standby
- set restore_command parameter to retrieve an archived segment of the wal file series.
  EX: restore_command = 'copy "C:\\archiveDir\\%f" "%p"
- set archive_cleanup_command to cleaning up old archived WAL files that are no longer needed by the standby server.
  - EX: archive_cleanup_command='pg_archivecleanup /mnt/server/archivedir %r'
- create standby.signal file in data directory. It informs the server to stay in recovery mode and continiously recover.
    When the file is removed, the server will open the database in read/write mode.
