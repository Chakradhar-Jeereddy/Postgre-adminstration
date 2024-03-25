# Repmgr & Automatic failover

- Repmgr is an open-source tool suite for managing replication and failover in a cluster of PostgreSQL servers.
- It supports and enhances PostgreSQL's built-in streaming replication.
- Repmgr provides single read/write primary server and on or more read-only standbys containing near-real time copies of the
   the primary server's database.
- Repmgrd deamon is user to actively monitor serversin a replication cluster.
- It is available only for Unix and linux.

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

## Repmgr Terminologies
- Replication cluster
   - Refers to the network of PostgreSQL servers connected by streaming replication.
- Node
   - A node is a single PostgreSQL server within a replication cluster.
- Upstream node
   - The node a standby server connects to, in order to receive streaming replication.
   This is either the primary server, or in the case of cascading replication, another standby.
- Failover
   - This is the action which occurs if a primary server fails and a suitable standby is promoted as the new primary.
   - The repmgrd deamon supports autmatic failover to minimize downtime.
- Switchover
   - In certain circumstances, such as hardware or operating system maintenance, it's necessary to take primary server
     offline; in this case a controlled switchover is necessary, whereby a suitable standby is promoted and existing
     primary removed from the replication cluster ina controlled manner. The repmgr command line client provides this
     functionality.
- Witness server
    - repmgr provides functionality to set up a so-called "witness server" to assist in determining a new primary server
  in a failover situation with more than one standby. The witness server itself is not part of the replication cluster,
  although it does contain a copy of the repmgr metadata schema.

   
