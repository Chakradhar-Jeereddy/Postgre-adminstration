* The maximum number of concurrent connections to the database server.
* The default is typically 100 connections. 97 for users and 3 for supper users.
* Set to the maximum number of connections that we expect to be need at peak load.
* Each connection uses shared_buffer memory, as well as additional non-shared memory
* Once the limit is reached application will throw “system out of memory” error.
* A connection can be active and doing some work or it can be idle.
* Idle connection refers to a connection that has been established between a client application and
  the database server but is not currently executing any queries or transactions.
* Connection can be idle if client application opens a connection but does not immediately execute any queries,
  or when a query or transaction is completed but the connection is not explicitly closed by the client.
* Idle connections can have an impact on the performance and scalability of a PostgreSQL database,
  as they consume server resources such as memory and CPU time, even though they are not actively processing
  any queries.

  ### Recommendations
* Pg_bench can be used to run benchmark test with varying workload models and multiple concurrent database sessions
* Average Transaction rate along with System resources usage like CPU and Memory Can be used in conjuction to arrive at acceptable number.
* Connection pooling can be used to increase and effectively handle postgresql connections.
* Pg_stat_activity is system view that stores PostgreSQL connection & activity stats.
* We can use the above view to find number of connections (Active & Idle Connections).





