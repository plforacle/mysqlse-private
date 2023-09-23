# MySQL Replication

## Introduction

Replication enables data from one MySQL database server (known as a source) to be copied to one or more MySQL database servers (known as replicas). 

In this lab you will prepare and create a replica.

Estimated Lab Time: 30 minutes

### Objectives
In this lab, you will:
* Install binaries on a new server
* Have that server act as a Replica


## Task 1: Prepare Replica

> **Note:**
 * Server: mysql2
 * mysql2 doesn't have binaries installed, so first part of the lab installs them. This is the same to first installation of mysql-advanced on mysql1

1. If instance mysql2 is not started, start it from OCI dashboard
2. Open an SSH client to app-srv
    ```
    <span style="color:green">shell></span><copy>ssh -i <private_key_file> opc@<your_compute_instance_ip></copy>
    ```
3. Connect to <span style="color:red">mysql2</span> server through app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql2 opc@mysql2</copy>
    ```
4. Execute below script that replicate what we did in manual installation lab (create mysqluser/mysqlgrp, folders and install binaries)
    ```
    <span style="color:green">shell-mysql2></span><copy>/workshop/support/MySQL_Replication___Prepare_replica_server.sh</copy>
    ```
5. <span style="color:red">Close and reopen the SSH connection</span> to let opc user have the new group.

## Task 2: Create Replica
> **Note:**
 * Servers: 
    * mysql1 (source)
    * mysql2 (replica)
 * MySQL 8.0 replication requires SSL. To make it works like MySQL 5.7 and practice for the exam we force the usage of old native password authentication in my.cnf
 * Some commands must run inside the source, other on replica: please read carefully the instructions

1. If not already connected, open two connections, one to mysql1 and one to mysql2
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```

    And in a second shell
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql2 opc@mysql2</copy>
    ```

2. <span style="color:red">mysql1 (source):</span> Create a backup of source in the shared folder to easily restore on the replica:
    * take a full backup of the source using MySQL Enterprise Backup in your NFS folder /workshop/backups:
    ```
    <span style="color:green">shell-mysql1></span><copy>mysqlbackup --port=3307 --host=127.0.0.1 --protocol=tcp --user=mysqlbackup --password --backup-dir=/workshop/backups/mysql1_source backup</copy>
    ```
3. <span style="color:red">mysql2 (replica):</span> We now create the my.cnf, restore the backup create and configure the replica
    * It’s mandatory that each server in a replication topology have a unique server id. There is a copy of the my.cnf ready to be used. It’s a duplicate of the one used for mysql-advanced instance, with a different server_id
    ```
    <span style="color:green">shell-mysql2></span><copy>sudo cp /workshop/support/my.cnf.mysql2 /mysql/etc/my.cnf</copy>
    ```
    ```
    <span style="color:green">shell-mysql2></span><copy>sudo chown mysqluser:mysqlgrp /mysql/etc/my.cnf</copy>
    ```
    * restore the backup from share folder
    ```
    <span style="color:green">shell-mysql2></span><copy>sudo /mysql/mysql-latest/bin/mysqlbackup --defaults-file=/mysql/etc/my.cnf --backup-dir=/workshop/backups/mysql1_source --datadir=/mysql/data --log-bin=/mysql/binlog/binlog copy-back-and-apply-log</copy>
    ```
    ```
    <span style="color:green">shell-mysql2></span><copy>sudo chown -R mysqluser:mysqlgrp /mysql</copy>
    ```

    * start the new replica instance it and verify that it works (hostname have to be mysql2).
    
        Don’t make any change to the instance content!
        ```
        <span style="color:green">shell-mysql2></span><copy>sudo systemctl start mysqld-advanced</copy>
        ```
        ```
        <span style="color:green">shell-mysql2></span><copy>mysql -uadmin -p -hmysql2 -P3307</copy>
        ```
        ```
        <span style="color:blue">mysql-replica></span><copy>SELECT @@hostname;</copy>
        ```
        ```
        <span style="color:blue">mysql-replica></span><copy>SHOW DATABASES;</copy>
        ```
4. <span style="color:red">mysql1 (source):</span> Create the replication user
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uadmin -p -hmysql1 -P3307</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>CREATE USER 'repl'@'%' IDENTIFIED BY 'Welcome1!' REQUIRE SSL;</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';</copy>
    ```

5. <span style="color:red">mysql2 (replica):</span> Time to connect and start the replica
    * Configure the replica connection.
    ```
    <span style="color:blue">mysql-replica></span><copy>CHANGE REPLICATION SOURCE TO SOURCE_HOST='mysql1', SOURCE_PORT=3307,SOURCE_USER='repl', SOURCE_PASSWORD='Welcome1!', SOURCE_AUTO_POSITION=1, SOURCE_SSL=1;</copy>
    ```

    * Start the replica threads
    ```
    <span style="color:blue">mysql-replica></span><copy>START REPLICA;</copy>
    ```

    * Verify replica status, e.g. that IO\_Thread and SQL\_Thread are started searching the value with the following command (in case of problems check error log)
    ```
    <span style="color:blue">mysql-replica></span><copy>SHOW REPLICA STATUS\G</copy>
    ```
6. <span style="color:red">mysql1 (source):</span> Let’s test that data are replicated. Connect to source and make some changes
    ```
    <span style="color:blue">mysql-source></span><copy>CREATE DATABASE newdb;</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>USE newdb;</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>CREATE TABLE t1 (c1 int primary key);</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>INSERT INTO t1 VALUES(1);</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>INSERT INTO t1 VALUES(2);</copy>
    ```
    ```
    <span style="color:blue">mysql-source></span><copy>DROP DATABASE employees;</copy>
    ```
7. <span style="color:red">mysql2 (replica):</span> Verify that the new database and table is on the replica, to do so connect to replica and submit
    ```
    <span style="color:blue">mysql-replica></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql-replica></span><copy>SELECT * FROM newdb.t1;</copy>
    ```

## Acknowledgements
* **Author** - Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
