```
base*           - database and objects are stored in this directory.
current_lofiles - Stores the location of the current log files.
global*         - It is used to store cluster wide objects such as users, roles,databases,tablespaces and toast tables.
log*            - Default directory to store logs, it can be changed using the parameter log_directory without restart.
pg_commit_ts*   - Stores information about commited transactions such as timestamps.
pg_dynshmem     - This directory contains the files used by the dynamic shared memory subsystem.
pg_logical      - This directory contains the status data of logical decoding when logical replication is implemented.
pg_multixact    - This directory contains multi-transaction status data.
pg_notify*      - This directory contains LISTEN/NOTIFY status data.
pg_replsot      - This directory contains replication slot data.
pg_serial       - This contains information about committed serializable transactions.
pg_snapshots    - This contains exported snapshots.
pg_stat         - This contains files for the statistics subsystem.
pg_stat_tmp     - This contains the temporary files for the statistics subsystem.
pg_subtrans     - This contains the subtransaction status data.
pg_tblspc       - This contains symbolic links to the actual tablespace directories.
pg_twophase     - This contains the state files for the prepared transactions.
pg_wal*         - This contains the transaction logs (write-ahead logs) of the PostgreSQL cluster, which are used for durability.
pg_xact         - This contains the transaction's commit status data.
pg_hba.conf     - This is the host-based authentication file that contains crucial data needed for client authentication. This file acts as a firewall to allow/restrict
connections from remote hosts to the database server.
pg_ident.conf   - PostgreSQL provides Ident-based authentication. It uses the client's operating system username as the allowed database username with an
operating system username mapping.
PG_VERSION      - This file contains the major version number of PostgreSQL that the data directory has been initialized with.
postgresql.conf - This file is the main configuration file used by PostgreSQL. The location may default to the data directory in RedHat/CentOS distributions
but changes to /etc/postgresql/13/main in Ubuntu/Debian distributions. This file contains all the parameters used to change PostgreSQL's behavior.
postgresql.auto.conf: This file contains the parameters that have been modified using ALTER SYSTEM in PostgreSQL. When PostgreSQL is started,
this is the last configuration read by PostgreSQL. So, if a parameter has two different values in the postgresql.conf and postgresql.auto.conf files,
the value that's set in the postgresql.auto.conf file is considered by PostgreSQL.
postmaster.opts - This file contains the command that's used to start the PostgreSQL cluster.
postmaster.pid* - This file contains data such as the process ID of the postmaster process, the data directory's location, port numbers, and the
connection's acceptance state of the cluster.
```
