```
TYPE       DATABASE       USER    ADDRESS   METHOD
local        all         postgres           peer
local        all         walle              peer

host      replication    pg       0.0.0.0/0   SCRAM-SHA-256
host      replication    gdat     0.0.0.0/0   SCRAM-SHA-256
host      telegraf      telegraf  170.90.0.123/32       SCRAM-SHA-256  
host      all            postgres           SCRAM-SHA-256
local     all            ldoadm02           peer

hostssl   all            +ldap_read,+ldap_readwrite,+ldap_readwritefull,+ldap_admin,+ldap_execute,+ldap_monitor,ldap_dba 0.0.0.0/0  ldap ldapserver="host", ldapport="1389",ldaptls='1',ldapprefix="uid=",ldap_suffix=",ou=Users,ou=Accounts,sc=ldo-p-0-us,dc-dn20,dc=scogen"
hostssl   replication    repmgr   primary-host  SCRAM-SHA-256
hostssl   replication    repmgr   secondary-host  SCRAM-SHA-256   
hostssl   all            all     0.0.0.0/0      SCRAM-SHA-256

TO check whether a connection is ssl or not use the view
select * from pg_stat_ssl;
pid  ssl   version     cipher                   bits        client_dn  client_serial  isser_dn
142   t     TLSc1.3  TLS_AES_256_OCM_SHA384      256
```

Note - 
For ldap, on client machine that is postgresql, sssd should be running.
User must be created in ldap server with password and on database also a user without password.
While connection, the user is verfied in ldap and authenticated (external authentication)
