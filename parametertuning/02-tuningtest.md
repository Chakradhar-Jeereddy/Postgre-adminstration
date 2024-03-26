# Introduction to Pg_Bench and Setup.
### Steps:
- Create a sample database ‘perftune’;
- Setup pgbench for the perftune database.
```
Pgbench -i -s 25 perftune
              -i = initialize 
              -s = scaling
             Pgbench by default create 16mb database,So we are scaling it with option 25. To create a database 25 times bigger than the default size (16 * 25) =400 MB
```
- Start creating baseline with the default out of the box parameters
- How to run a test
```
pgbench -c 10 -j 2 -t 10000

-c =Client
-j  = threads 
-t  = transactions
-S = select only queries
-n = skip vacuuming
```
- What is the "Transaction" Actually Performed in pgbench?
- The default transaction script issues seven commands per transaction:
```
1. BEGIN;
2. UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
3. SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
4. UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
5. UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
6. INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
7. END;
If you specify -N, steps 4 and 5 aren't included in the transaction. If you specify -S, only the SELECT is issued.
```
