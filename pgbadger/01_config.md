- The tool reads the logs and generate reports.

Step by Step Installation and Reporting:

Download pgbadger:
https://github.com/darold/pgbadger/releases

prerequisites:
yum install perl-devel

and on RPM-like system using:
    sudo yum install perl-JSON-XS - sudo yum install perl-text-csv_xs


1) tar -xzf pgbadger-12.4.tar.gz
2) cd pgbadger-12.4
3) perl Makefile.PL

[postgres@standby pgbadger-12.4]$ perl Makefile.PL
Checking if your kit is complete...
Looks good
Generating a Unix-style Makefile
Writing Makefile for pgBadger
Writing MYMETA.yml and MYMETA.json

4) make && sudo make install

5) Setup Environment Variable
LD_LIBRARY_PATH=/usr/pgsql-16/lib
export LD_LIBRARY_PATH

PATH=/usr/pgsql-16/bin:$PATH
export PATH

export PGDATA='/var/lib/pgsql/16/data'
export PGLOG='/var/lib/pgsql/16/data/log'
export PGBADGER='/var/lib/pgsql/16/pgbadger/pgbadger-12.4'
export PGREPORTS='/var/lib/pgsql/16/reports'

./pgbadger --help

6) psql
Log settings:
alter system set logging_collector = 'on';
alter system set log_truncate_on_rotation = 'on';
alter system set log_rotation_age = 1440;
alter system set log_filename = 'postgresql-%a.log';

What to log:
alter system set log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a,client=%h';
alter system set log_checkpoints ='on';
alter system set log_connections ='on';
alter system set log_disconnections ='on';
alter system set log_lock_waits ='on';
alter system set log_temp_files = 0;
alter system set log_autovacuum_min_duration = 0;
alter system set log_min_duration_statement='50ms';
alter system set deadlock_timeout = '1s';
alter system set log_error_verbosity = 'terse';
SELECT pg_reload_conf();

7) Generate workload(optional)
pgbench -c 10 -j 2 -t 1000 postgres

8) Reports:

./pgbadger /var/lib/pgsql/16/data/log/postgres* -O /var/lib/pgsql/16/reports

with filename:
./pgbadger  /var/lib/pgsql/16/data/log/postgres* -o /var/lib/pgsql/16/reports/today_report.html


Error Reporting
./pgbadger -q -w  /var/lib/pgsql/16/data/log/postgres* -o /var/lib/pgsql/16/reports/today_report.html


To incrementally analyze logs and add the results to a single report, use the --last-parsed and --outfile options.

./pgbadger /var/lib/pgsql/16/data/log/postgresql*.log --last-parsed /var/lib/pgsql/16/reports/pgbadger_last_state_file --outfile /var/lib/pgsql/16/reports/report.html

Cron- Auto incremental pgbadger reports on weekly basis.

  0 4 * * * /var/lib/pgsql/16/pgbadger/pgbadger-12.4/pgbadger -I -q /var/lib/pgsql/16/data/log/postgresql*.log -O /var/lib/pgsql/16/reports/


By default, PgBadger will generate an HTML report. However, you can also choose from other output formats (like CSV or JSON) using the --format option.

pgbadger /path/to/postgresql.log -o report.csv --format csv

