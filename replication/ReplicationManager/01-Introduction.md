# Repmgr & Automatic failure

- Repmgr is an open-source tool suite for managing replication and failover in a cluster of PostgreSQL servers.
- It supports and enhances PostgreSQL's built-in streaming replication.
- Repmgr provides single read/write primary server and on or more read-only standbys containing near-real time copies of the
   the primary server's database.
- Repmgrd deamon is user to actively monitor serversin a replication cluster.

## Commponents of Repmgr
- Repmgr: A commmand-line tool used to perform administrative tasks such as:
    - Setting up standby servers
    - Promoting a standby server to primary
    - Switching over primary and standby servers
    - Display the status of servers in the replication cluster.

  - Repmgrd: Deamon which actively monitors servers in a replication cluster and performs the following tasks.
    - Monitoring and recording replication performance.
    - Performing failover by detecting failure of the primary and promoting the most suitable standby server.
    - Provide notifications about envents in the cluster to a user-defined script which can perform tasks such as sending
       alert by email.
