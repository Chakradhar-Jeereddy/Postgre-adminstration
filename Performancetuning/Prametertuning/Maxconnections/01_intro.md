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

## How to affectively use connections using PGQL parameters

* idle_session_timeout: Interactive connections (Pgadmin,psql and so on), not for applications using connection pooling, in connection pooling connections are pre-established and they remain idle and reused to when a user request arives and reduces the connection time.
* We should not set this parameter at cluster level, set it only at user level. Like user profile in oracle.
```
alter user postgres set idle_session_timeout='5min';
alter user postgres set idle_transcation_session_timeout='5s';  -10ses
Conservative go for 10mins, want to be aggressive go for 5 mins
```
```
At session level
postgres=# set idle_session_timeout to '700';
SET
postgres=# select * from pg_stat_activity;
FATAL:  terminating connection due to idle-session timeout
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
postgres=#
```

* idle_in_transaction_session_timeout: Also for interactive users. If you have transaction with multiple statements, first statement is done and then there is long wait time for second operation or statement is not getting executed, it is waiting. It being but not moving further. Set the value very low in millie seconds.
```
postgres=# set idle_in_transaction_session_timeout to '3000';
SET
postgres=# begin;
BEGIN
postgres=*# select pg_sleep(5);
 pg_sleep
----------

(1 row)

postgres=*# select * from pg_stat_activity;
FATAL:  terminating connection due to idle-in-transaction timeout
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.

```
### Below parameters can be set at cluster level
* tcp_keeplives_idle
* tcp_keeplives_interval
* tcp_keeplives_count
* client_connection_check_interval
* Authentication_timeout

* tcp_keeplives_idle = 300(seconds) - This is network parameter, if there is no communication between client and server, after three seconds it will sent a network packet asking are you available and wait for response from client.
* tcp_keepalives_interval = 20 seconds - After 20 seconds it will retry
* tcp_keeplives_count = 5, after 5 seconds it declare the connection/session is terminated
* client_connection_check_interval = 5 seconds, it keeps checking if client as gone away are not if not gone, it will be terminated.
* Authentication_timeout - Determine how long the client authenticaton should take.

