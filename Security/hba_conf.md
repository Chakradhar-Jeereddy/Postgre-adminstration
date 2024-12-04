TYPE       DATABASE       USER    ADDRESS   METHOD
local        all         postgres           peer
host      replication    pg       0.0.0.0/0   SCRAM-SHA-256
host      replication    gdat     0.0.0.0/0   SCRAM-SHA-256
host      telegraf      telegrag  ip/32     trust
host      all            postgres           SCRAM-SHA-256
local     all            ldoadm02           peer
hostssl   replication    repmgr   primary-host  SCRAM-SHA-256
hostssl   replication    repmgr   secondary-host  SCRAM-SHA-256   
hostssl   all            all     0.0.0.0/0      SCRAM-SHA-256

TO check whether a connection is ssl or not use the view
select * from pg_stat_ssl;
pid  ssl   version     cipher                   bits        client_dn  client_serial  isser_dn
142   t     TLSc1.3  TLS_AES_256_OCM_SHA384      256
