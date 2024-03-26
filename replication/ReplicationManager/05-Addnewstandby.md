# Case 3: Add new standby and standby follow

## Objective: To build a new standby server with primary backup and how to make standby server follow
   New primary in case of failover.

## Instance : 
Primary : 192.168.1.8
Standby 1: 192.168.1.9
Standby 2: 192.168.1.5

## Steps: 
1)	Run a dry run to test the configuration.
```
repmgr -h 192.168.1.8 -U repmgr -f /var/lib/pgsql/repmgr.conf standby clone –dry-run
```
3)	 Start the clone of standby server
```
/usr/pgsql-12/bin/repmgr -h 192.168.1.8 -U repmgr -f /var/lib/pgsql/repmgr.conf standby clone
```
5)	Start postgresql on standby server.
6)	Register standby server with repmgr.
```
 Cd /usr/pgsql-12/bin
./repmgr -f /var/lib/pgsql/repmgr.conf standby register
```
8)	Verify whether standby server is register with repmgr and standby is following primary.
```
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show.
```
10)	Ensure repmgrd is running on all the servers.
11)	Turn off primary database and initiate failover.
12)	Check the new standby as primary.
13)	Initiate the second standby to follow
```
./repmgr -f /var/lib/pgsql/repmgr.conf standby follow
``` 
14)	Verify whether standby server is register with repmgr and standby is following the new primary.
```
./repmgr -f /var/lib/pgsql/repmgr.conf cluster show
```
