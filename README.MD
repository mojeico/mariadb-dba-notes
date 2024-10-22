




Command:


status  - status db


system df - run system command from sql

use - change datadabe


- system vi test.sql - create sql fiel
- source test.sql 	 - run sql file










GLOBAL VARIABLES - (after restart will be changed to default )

- show global variables;
- show global variables like 'max_connections';
- show global variables like 'max_%';


- set global max_connections=300; -










SESSION VARIABLE  - only in current session



-  show session variables like 'max_%';











SYSTEM VARIABLE

https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html












SHOW COMMAND

- show databases;
- show processlist; - all connections
- show events; 	- all events in database, not in server
- show tables;












SYSTEM DATABASES



- imformation_chema (system catalog)-  access to metadata, data about data  - read only
		- select * from processlist; == show processlist;
		- select * from engines;


- mysql -  about mysql server
 		- select user,host from user;

- performance_schema - infomation about execution of the server, memory allocation and etc ...
		- select * from file_instances; - show all mariadb server files

- sys - looks like the same as performance_schema but with ather information
		- select * from host_summary;













USERS:

- use mysql;
- desc user;
- select user,host,password from user;











MARIADB-SHELL

https://mariadb.com/docs/server/connect/clients/mariadb-shell/
yum install mariadb-shell









MARIADB-ADMIN



- mariadb-admin create test1 - create db





RUN FILES IN MARIDB



- touch test.sql


- source test.sql
- cat test.sql | mariadb






RUN COMMAND

- mariadb -e "show databases;"
- mariadb -e " show databases; show processlist; "






MARIADB CHECK  - mariadb-check - check table - table will be locked



- mariadb-check mysql db







MARIADB SHOW

- mariadb-show mysql user host

- mariadb-show db_name table_name column_name





MARIADB BIN LOG

https://stackoverflow.com/questions/11445678/binary-log-error-in-mysql
add log-bin=mysql-bin in  /etc/my.cnf.d/server.cnf


- mariadb-binlog
- show binary logs;








############################################################################### SHOW STORAGE ENGINES ###############################################################################


MARIADB ENGINS
- show engines;





ENGINES

- show engines;

- show variable like '%plugin%';





MEMORY ENGINE  - keep in memory, very fast, for temporary tables, after restart is empty

- CREATE TABLE continents ( cid int NOT NULL, cname VARCHAR(25) NOT NULL) ENGINE=MEMORY;


- SELECT TABLE_NAME,TABLE_TYPE,ENGINE,TABLE_ROWS,CREATE_TIME FROM INFORMATION_SCHEMA.TABLES WHERE ENGINE='MEMORY'\G;






CHECK STATUS ENGINE

- show engine innodb status ;




INSTALL NEW ENGINE\PLUGIN

- check
	[mysqld]
	plugin_dir=/path/to/plugin/directory

-  INSTALL PLUGIN federated  SONAME 'ha_federated.so';





############################################################################### USERS ADMINISTRATION ###############################################################################





Create user


- create user if not exists 'username_dba' identified by 'password' password expire never account unlock;

	- username_dba - username
	- with password 'password'


- grant all privileges on *.* to 'username_dba'  with grant option;

	- *.* to all db, tables and etc
	- with grant option - can give grant to another user


- grant select on mysql.* to gleb;



- flush privileges; ?



- show grants for username_dba; - show all permission



- select user(),current_user();








mysql_native_password & caching_sha2_password auth plugins














ROLE


- create role if not exist reader;
- create role if not exists writer;
- create role if not exists admin;


- grant select on mysql.* to reader;
- grant insert, update, delete on mysql.* to writer;
- grant all  on mysql.* to admin;



- show grants for reader;
- show grants for writer;
- show grants for admin;



-  grant admin to gleb;



- flush privileges;






###############################################################################. SERVER CONFIGURATION #############################################


mariadbd --verbose --help - check all options


cat /etc/my.cnf.d/server.cnf - default configuration file 

cat /etc/my.cnf



strace mariadb -  The ‘strace’ command, akin to a detective, can assist you in tracing system calls and signals in your Linux system

lsof -u mysql - Check all files opened by mysql 








ls  /var/lib/mysql/    - keep data directory 


Move  data directory 

	 - stop mariadb - systemctl stop mariadb
     - mv * prod
     - chown -R mysql:mysql  prod
	 - add "datadir = /var/lib/mysql/prod" in /etc/my.cnf.d/server.cnf (mysqld) 





binary log - record database events and changes  - allow point-in-time recovery - 1GB
/var/lib/mysql/mysql-bin.000001
/var/lib/mysql/mysql-bin.index


- show binary logs; - show files and size 
- show binlog events in 'mysql-bin.000001' ; - show all events in binlog file 
- purge binary logs to 'mysql-bin.000004'; - delete binary logs from 0 to 4 
- mariadb-binlog 






show variables like "%error%"; -> log_error


- touch  /var/log/mariadb/errorlog/error.log - create dir and file
-  chown -R mysql:mysql  /var/log/mariadb
- log-error = /var/log/mariadb/errorlog/error.log - add in /etc/my.cnf.d/server.cnf (mysqld)







ls -l /var/lib/mysql/
ls -l /usr/bin/mysql*



###############################################################################. INNODB STORAGE CONFIGURATION  #############################################



INNODB BUFFER POOL - table cache and index data


show engine innodb status
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';


- add innodb-buffer-pool-size = 25M in /etc/my.cnf.d/server.cnf (mysqld)







INNODB LOG BUFFER - holds data to be written to log file - allow transactions to run without commiting


- show global variables like '%innodb_log_buffer_size%';
- show global status like 'innodb_log_waits'; - The number of times that the log buffer was too small and a wait was required for it to be flushed before continuing. If more than 10, have to increase innodb_log_buffer_size




INNODB FLUSH METHOD - determines how data flushes (move) from memory to disk



show global variables like "innodb_flush_method"; == mariadbd  --verbose --help | grep innodb-flush-method



- add innodb-flush-method = O_DIRECT in /etc/my.cnf.d/server.cnf (mysqld)




DOUBLEWRITER BUFFER

- READ MORE INFORMATION





INNODB REDO LOG FILES - about data recovery - for crash recovery -disk data structure - using for correcting data written by incomplete transactions

/var/lib/mysql/ib_logfile0

- READ MORE INFORMATION




SYSTEM TABLESPACE - storage aria for change buffer
file: ibdata1 (in datadir)



UNDO TABLESPACE - information how to undo the latest changes by trx

file: undo001,undo002,.... (in datadir)










DEDICATED SERVER - use all resource of instance


show global variables like '%dedicated%';






###############################################################################. BACKUP & RECOVERY   #############################################



PHYSICAL / COLD BACKUP - copy instance files (no backup tool is required) - when database OFFLINE

1) shutdown mysql
2) copy all data derectory(user-space), system tablespace(system-space) and option files (any *.cnf files )



- show global variables like "datadir"; - (user-space)
- show global variables like "innodb_data_home_dir"; - (system-space)



LOGICAL BACKUP - only data(database and tables) backup by dump or pump, generate sql file


- mariadb-dump --all-databases > a.sql
- mariadb < a.sql   - restore



HOT BACKUP -  copy instance files (no backup tool is required) - when database ONLINE - xtrabackup or  mariadb-backup


- yum install MariaDB-backup


- yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
- sudo percona-release enable-only tools release
- sudo yum install percona-xtrabackup-80



- xtrabackup --backup --user=username_dba --host=localhost --port=3306 --password=password  --target-dir=~/backup



-  mariadb-backup ----- need to read article



- mariadb  -u username_dba -p (password)











###############################################################################. SERVER REPLICATION   #############################################



METHODS

- Traditional - base on binary log file
	- replication events from primary binary log file
- GTID - global transactions indetifier
 	- Newer method, not work with binary log file
	- transactions apply to the replica
 	- Recomended way




TRADITIONAL METHOD BASE ON BINARY LOG FOR REPLICAS ;


-------- MASTER ----------
- Enable binary log

		log_bin_index     -  /var/lib/mysql/mysql-bin.index
        log_bin_basename  -  /var/lib/mysql/prod/mysql-bin
	    log_bin           -  ON

- Set server id

		server_id         -  1

- Create user for replication user

		CREATE USER IF NOT EXISTS replicator IDENTIFIED BY 'password';
		GRANT REPLICATION SLAVE ON *.* TO replicator;
		FLUSH PRIVILEGES;

- Set binary log format

		binlog_format  	  - MIXED

- Check master status

		 SHOW MASTER STATUS;


-------- REPLICA --------

- Enable relay log


		relay-log  = /var/lib/mysql/mysql-relay
		relay-log-index  = /var/lib/mysql/mysql-relay.index

		read-only

		skip-slave-start

- Set server id

		server_id         -  2

- Change mastr statement

	    change master to master_host='10.214.0.2', master_user='replicator', master_password='password', master_port=3306, master_log_file='mysql-bin.000011', master_log_pos=342;

- start replica

		start replica;
		show replica status\G;

		in master - SHOW replica  HOSTS;





GTID METHOD FOR REPLICAS ;

#
# after all course cehck it again in Udemy and recreate VM in GCP
#







###############################################################################. MAX SCALE    #############################################













































































































































































































































































































































