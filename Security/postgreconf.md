```
listen_addesses = '*'
port = 12400
unix_socket_directories = '/var/run/postgresql'
max_connections=100
superuser_reserved_connections = 3
password_encryption= scram-sha-256
logging_collector=on
log_filename=
log_line_prefix=
log_directory=
log_file_mode=0640 for tools that need to check logfile
log_truncate_on_rotation=on
log_rotation_age=1d
log_truncate_on_rotation=on
log_rotation_age = 1d
log_timezone='UTC'
log_lock_waits= on
log_checkpoints = on
log_connections = on
log_disconnections = on
log_hostname = off
log_autovaccum_min_duration = 5000
```
ssl = on
ssl_cert_file = ''
ssl_key_file= ''
log_destination= 'stderr, syslog'
pgaudit.log = 'all,-mic,-read,-write,-function,-ddl'
pgaudit_log_parameter = on

archive_mode=on
archive_timeout=900  => wether or not a wall is filled it defines the length of time(seconds) the data remain un archived. Used for systems which not very busy
wal_level=hot_standby  (depereciared, should use replica) => allowed values are minmal, replica, logical (minal - log only data required for crash recovery, replica - data recquired for poin in time, streams and continus archiving)
                                                            and max_wal_senders must be non zero and logical for logical replication)
archive_command = 'pgbackrest --stanza=PGCLUSTER archive_push %p'
max_wal_senders=10
hot_standby = on (run read only commands)
max_replication_solts = 6


