Transaction wrap arround

```
select count(*) from pg_prepared_xacts;
select count(*) from pg_stat_replication;
```

What is transaction wraparound -?

In postgresql when every transaction performed, it will use 32 bit transaction id and these transaction ID has some limit(4 billion), When max number are reused
Postgres will start using the transaction ID from the begining.

When it happens?
Autovacuum is turned off
Heavy DML operation, autovacuum gettings canceled.
Long running transaction
Many session or connections holding lock for long time.

How to identify?
When postgre reach 2 bilion un-vacuumed transactions, the cluster gets into read only mode and will not allow writes to privent data loss.
Does not allow writes into database.

1) Postgres won't allow operaions in DB.
2) In Postgres DB log you will find below error msg. Error: Database is not accepting commands to avoid wrap around data loss in postgres.

How to prevent - 
SELECT datname,
age(datfrozenxid),
100*(age(datfrozenxid)/current_setting('autovacuum_freeze_max_age')::float) AS emergency_autovacuum_percent,
(age(datfrozenxid)::numeric/2000000000*100)::numeric(4,2) as "wraparound_risk_percent"
FROM pg_database ORDER BY 2 DESC


Check table wise auto-vacuum
SELECT datname,
age(datfrozenxid),
100*(age(datfrozenxid)/current_setting('autovacuum_freeze_max_age')::float) AS emergency_autovacuum_percent,
(age(datfrozenxid)::numeric/2000000000*100)::numeric(4,2) as "wraparound_risk_percent"
FROM pg_database ORDER BY 2 DESC

warparound_risk_percent = age(datforzenxid/2 bilion*100) = Put an alert at 75-80% warparound risk percentage.

Run manual vacuum periodically on heavy transaction DB.

If we encounter wrap arround situation- 

Steps to fix
Check database size
Check disk allocated to host
Additional space to perform full vacuum
stop postgres
get into single user mode
run full vacuum on each database.
exit singe mode
start postgres
connect to db
create sample table.








