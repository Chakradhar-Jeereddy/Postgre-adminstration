Application Failover using libpq Features in PostgreSQL:
libpq is the C application programmer's interface to PostgreSQL. libpq is a set of library functions that allow client programs to pass queries to the PostgreSQL backend server and to receive the results of these queries.
libpq is also the underlying engine for several other PostgreSQL application interfaces, including those written for C++, Perl, Python, Tcl and ECPG.
Starting from PostgreSQL 10, libpq  and psql clients could probe the connection for a master and allow connections to a master for read-write or any node for read-only connections automatically.

Master : 192.168.1.8
Standby : 192.168.1.9
Standby1 : 192.168.1.5

Steps:
1)	Connect to Master node and query 
Select  inet_server_addr();
-------------------------------------------
192.168.1.8
inet_server_addr returns the IP address on which the server accepted the current connection

2)	So now I use psql to probe all my server address which are open for read write request. As we know only the master will accept read/write connection and others are  in read only mode. Run the below mentioned query in all the nodes in cluster and it will tell you which node is master and which one is standby.
$ psql 'postgres://192.168.1.8:5432,192.168.1.9:5432,192.168.1.5:5432/postgres?target_session_attrs=read-write' -c "select inet_server_addr()"
3)	Now I want to find which all nodes in the cluster are in read only mode, I will use the same psql command by changing only the target_session_attrs to ‘any’ and probe the read only nodes.
$ psql 'postgres://192.168.1.8:5432,192.168.1.9:5432,192.168.1.5:5432/postgres?target_session_attrs=any' -c "select inet_server_addr()"
So how does it work.
1)	When I specify all my nodes ip address in the connection string and try to connect, depending on pg_is_in_recovery status. Which ideally is either ‘True’ or ‘False’, it decides whether a node is read/write or read only. 
2)	If my primary/master fails my the system wont be available in the cluster and the connect command will not be able to query that particular node.
3)	It will probe the other node is cluster and will find that one of the standby has taken the role of the primary, because pg_is_in_recovery status is ‘True’ in one of the standby.
4)	So all the read/write request will be transferred to that particular node.

Sample Python connection code 
$ cat pg_conn.py
import psycopg2
conn = psycopg2.connect(database="postgres",host="192.168.1.8,192.168.1.9,192.168.1.5", user="postgres", password="xxxxxx", port="5432", target_session_attrs="read-write")
cur = conn.cursor()
cur.execute("select pg_is_in_recovery(), inet_server_addr()")
row = cur.fetchone()
print "recovery =",row[0]
print "server =",row[1]
If I execute this file. I will come to know which server is my primary depending on pg_is_in_recovery_Status
$ python pg_conn.py
recovery = False     server = 192.168.1.8.
