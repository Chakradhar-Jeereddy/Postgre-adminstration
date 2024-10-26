-- Postgresql Source Installation 16.2
```
1)	Download the source file: https://www.postgresql.org/ftp/source/ 

2)	Prerequisites:
   yum install readline-devel
   yum install -y zlib-devel
   yum install -y install gcc
   yum install -y make
   yum install -y libicu-devel
      
3)	Extract tar file and check configure:
tar -xvf postgresql-16.2.tar.gz
cd postgresql-16.2
./configure –help

4)	Create postgres user and required directories:
useradd -d /home/postgres/ postgres
passwd postgres
id postgres
mkdir -p /u01/app/16.2/init
mkdir -p /u02/app/16.2/data
mkdir -p /u03/app/16.2/wal_files 
mkdir -p /u04/app/16.2/archive_logs
mkdir -p /u05/app/16.2/temp_files

5)	Configure PostgreSQL:
  cd postgresql-16.2
. /configure --prefix=/u01/app/16.2/init


6)	Build postgreSQL using make command:
 cd postgresql-16.2
 make 
or 
make world (everything including contrib, documentation and man pages) 
or 
make world-bin (everything, except documentation).
 
7)	Install postgreSQL using make install command:
make install 
or 
make install-world 
or 
make install-world-bin

8)	Build contrib module:
 cd contrib
 make

9)	Install contrib module: 
make install


10	) Check Bin Folder pg_config for directory structure:
          cd /u01/app/16.2/init/bin
         ./pg_config

    11) Change ownership from root to postgres:
chown -R postgres:postgres /u01/app/16.2/init
chown -R postgres:postgres /u02/app/16.2/data
chown -R postgres:postgres /u03/app/16.2/wal_files 
chown -R postgres:postgres /u04/app/16.2/archive_logs
chown -R postgres:postgres /u05/app/16.2/temp_files

12) Initialize postgreSQL data directory:
        sudo su - postgres
        cd /u01/app/16.2/init/bin
       ./initdb -D /u02/app/16.2/data

13) Check the content of DATA Directory and Start Database:
        cd /u02/app/16.2/data
          ls -ltr
         cd /u01/app/16.2/init/bin
         ./ pg_ctl -D   /u02/app/16.2/data start 

14)  Set Environment Variables and Connect to psql and check.
       export LD_LIBRARY_PATH=/u01/app/16.2/init/lib:$LD_LIBRARY_PATH
       export PATH=/u01/app/16.2/init/bin:$PATH
       postgres# psql
       
```


         
        


  
       


