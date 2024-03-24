## Failover:
- Shutdown postgresql on master and check whether it affects standby
- Test replication
```
psql> insert into master(2);
Error: connot execute insert in a read-only transactions
```
- Promote your standby database to primary. Use any of the 3 methods
```
Pg_ctl promote -D /var/lib/pgsql/12/data
```
- Make changes
```
psql> insert into master(2);
1 row inserted
---

- Check signal.standy exist or not.
- Copy hba.conf details to new master for applications to connect
- Rebuilt new standby and reenable replciation.
- Bring up the postgresql old master server and try to create a table there and check whether its replicated.
   - it won't
