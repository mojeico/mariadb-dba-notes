




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










ls -l /var/lib/mysql/
ls -l /usr/bin/mysql*




























































































































































































































































































































































































