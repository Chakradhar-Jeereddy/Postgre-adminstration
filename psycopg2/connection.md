### Psycopg2 is a popular postgresql adaptor for python that allows you to interact with postgresql databases.
```
Database operations that it can perform
* Query executions
* Managing transactions
* Handling connections
* Working with connections
```

```
Connection methos
* connect()
  It establish a connection to database and returns a connection object.

Code to get connection object
import psycopg2
conn = psycopg2.connect(dbname='',user='',port='',host='',password='')

Code to close connection
conn.close()

Code to commit
conn.commit()
conn.rollback()
conn.autocommit=True
```

```
To interact with database we need cursor object
cur = conn.cursor()
cur.execute("select * from users where id=()")
rows=cur.fetchall()
print(rows)

cur.fetchone()
cur.fetchmany(size)
cur.execute("create table it not exist dept")
cur.commit()

cur.close()
conn.close()
```

