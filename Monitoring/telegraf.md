```
Telegraf is an open source data collection agent.
Telegraf agent deployed in each VM hosting postgresql cluster.
It collects metrics every 60s and sends it kafka topics from there it goes to data lake.
The kafka broker, topic information is defined in the telegraf config file.
data_format - customjson
The mertics that are collected using input plugins:

Inputs.cpu
  percpu = true
  totalcpu = true
  collect_cpu_time = true
Inputs.mem

Inputs.dislio

Inputs.disk
ignore_fs = ["tmpfs","devtempfs","devfs"]

Inputs.netstat

Inputs.exec
 commands = ["/export/postgres/pg_status.ksh"]
systemctl status $(systemctl list-unit-files --state=enabled |grep postgres|grep -v telegraf|awk {'print $1'})|grep "Active : active (running)" > /dev/null 2>&1

Inputs.net

Inputs.postgresql
address ="host port user=telegraf password database=telegraf
ignore_database = [postgres, telegraf,template0,template1]

Inputs.postgresql_extensible
 address = "host, port,....
 sqlquery="select * from pg_stat_database"

select * from pg_stat_bgwriter;

max_connections

select setting from pg_settings where name="shared_buffers"
select blk_hit/blk_hit+blk_reads from pg_stat_database;

select nb_sess,act_sess, blk_sess from (select count(1) as nb_sess from pg_stat_activity) a, (select count(1) as act_sess from pg_stat_activity where status='active') b, (select count(1) as blk_sess from pg_stat_activity where wait_event not null and status='active') c;

select datname,pg_database_size(datname) from pg_database where datname not in (postgres,repmgr,telegraf,template0,template1);

select xact_commit/xact_commit+xact_rollback as commit_ratio from pg_stat_database;

select sent_lsn,flush_lsn,replay_lsn,max(pg_wal_lsn_diff(sent_lsn,replay_lsn)) as lag_in_bytes from pg_stat_replication;
```




