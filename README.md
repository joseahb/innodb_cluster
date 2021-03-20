# innodb_cluster
 
 Set up

- Three servers running ubuntu 20.04 LTS server
- Set hostnames in /etc/hosts in each of the hosts file to resolve IPs of nodes; 
    127.0.0.1 localhost
    192.X.X.X  hostname01
    192.X.X.X  hostname02
    192.X.X.X  hostname03
 

# Install mysql Server

-> Remove existing files

root> apt purge mysql*

root> rm /etc/mysql/ -r

root> rm /var/run/mysqld/ -r

# Install mysql server and Mysql Shell

<root># dpkg -i mysql-apt-config-XX.deb
<root># apt update
<root># apt install mysql-community-server mysql-shell

-> Go to <mysql> terminal

mysql> set sql_log_bin=0;

mysql> create user 'amdev'@'%' identified by 'qwerty';

mysql> GRANT ALL PRIVILEGES ON *.* TO 'amdev'@'localhost' identified by 'qwerty' with grant option;

mysql> GRANT ALL PRIVILEGES ON *.* TO 'amdev'@'%' identified by ‘qwerty′ with grant option;

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE, CREATE ROLE, DROP ROLE ON *.* TO `amdev`@`%` WITH GRANT OPTION;

mysql> GRANT APPLICATION_PASSWORD_ADMIN,AUDIT_ADMIN,BACKUP_ADMIN,BINLOG_ADMIN,BINLOG_ENCRYPTION_ADMIN,CLONE_ADMIN,CONNECTION_ADMIN,ENCRYPTION_KEY_ADMIN,FLUSH_OPTIMIZER_COSTS,FLUSH_STATUS,FLUSH_TABLES,FLUSH_USER_RESOURCES,GROUP_REPLICATION_ADMIN,INNODB_REDO_LOG_ARCHIVE,INNODB_REDO_LOG_ENABLE,PERSIST_RO_VARIABLES_ADMIN,REPLICATION_APPLIER,REPLICATION_SLAVE_ADMIN,RESOURCE_GROUP_ADMIN,RESOURCE_GROUP_USER,ROLE_ADMIN,SERVICE_CONNECTION_ADMIN,SESSION_VARIABLES_ADMIN,SET_USER_ID,SHOW_ROUTINE,SYSTEM_USER,SYSTEM_VARIABLES_ADMIN,TABLE_ENCRYPTION_ADMIN,XA_RECOVER_ADMIN ON *.* TO `amdev`@`%` WITH GRANT OPTION;

mysql> change master to master_user=’replamdev’, master_password=’qwerty′ for channel ‘group_replication_recovery’;

mysql> flush privileges;

mysql> set sql_log_bin =1;

mysql> reset master;

-> Login to mysql Shell

<root># mysqlsh -u amdev Hostname01

mysqlshJS> dba.configureLocalInstance('amdev@Hostname01:3306);//for all hosts

-> Repeat step above for all the nodes
-> Create cluster

mysqlshJS> dba.createCluster('myCluster');

mysqlshJS> cluster = dba.getCluster();

mysqlshJS> cluster.addInstance('amdev@Hostname02:3306);

mysqlshJS> cluster.addInstance('amdev@Hostname03:3306);

mysqlshJS> cluster.status();

-> All instances should have joined the cluster
alternatively check on mysql terminal using

mysql> select * from performance_schema.replication_group_members;

# Installing & configuring mysql router;

<root># apt update & upgrade
<root># apt install mysql-apt-config
<root># apt install mysql-router

# Start mysql-router 
<root># mysqlrouter --bootstrap amdev@Hostname01 -d myrouter --user=amdev

If errors occur when starting mysql router, check and add appropriate permissions in apparmor/sbin.mysqlrouter.

Now consume the database through the hostname of your router;

mysql -u amdev -h ROUTER_IP -P 6446 -e "select @@hostname" -p


