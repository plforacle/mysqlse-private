# Installation of MySQL Enterprise

## Introduction

In this lab you will install MySQL 8 Enterprise on Linux

Estimated Time: 30 minutes

### Objectives

In this lab, you will be guided trough the following tasks:
* Tarball installation of MySQL 8 Enterprise on Linux


> **Note:** 
  Server: mysql1
  * we call this instance <span style="color:red">mysql-advanced</span>
  * this is the most used instance in the labs


## Task 1: Install MySQL Enterprise

1. If not already connected, connect to mysql1 server trhougth app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy> ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1 </copy>
    ```
2. On Oracle Linux8/RHEL8/Centos 8 is required to install ncurses-compat-libs to use the tar package (not for the rpms)
    ```
    <span style="color:green">shell-mysql1></span><copy> sudo yum install -y ncurses-compat-libs </copy>
    ```
3. Usually, to run mysql is used the user “mysql”, but because he’s already available we show here how create a new one.
Create a new user/group for your MySQL service (mysqluser/mysqlgrp) and a add ‘mysqlgrp’ group to opc to help labs execution
    ```
    <span style="color:green">shell-mysql1></span><copy> sudo groupadd mysqlgrp </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy> sudo useradd -r -g mysqlgrp -s /bin/false mysqluser </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy> sudo usermod -a -G mysqlgrp opc </copy>
    ```
4. Create new directory structure:
    ```
    <span style="color:green">shell-mysql1></span><copy> sudo mkdir /mysql/ /mysql/etc /mysql/data </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo mkdir /mysql/log /mysql/temp /mysql/binlog </copy>
    ```
5. To simplify the lab, add the mysql bin folder to the bash profile and customize the client prompt. Please insert these lines at the end of the file /home/opc/.bashrc
    ```
    <copy>export PATH=$PATH:/mysql/mysql-latest/bin</copy>
    ```
    ```
    <copy>export MYSQL_PS1="\\u on \\h>\\_" </copy>
    ```
  You can edit the file with the editor that you prefer, here some examples: 
    ```
    <span style="color:green">shell-mysql1></span><copy>nano /home/opc/.bashrc </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>vi /home/opc/.bashrc </copy>
    ```
6. <span style="color:red">Close the ssh session and reopen it to activate the new privilege and settings for opc user</span>
7. Extract the tarball in your /mysql folder
    ```
    <span style="color:green">shell-mysql1></span><copy>cd /mysql/ </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo tar xvf /workshop/linux/mysql-commercial-8.0.*-linux-glibc2.12-x86_64.tar.xz</copy>
    ```
8. Create a symbolic link to mysql binary installation
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo ln -s mysql-commercial-8.0.*-linux-glibc2.12-x86_64 mysql-latest </copy>
    ```
9. Create a new configuration file <span style="color:red"> my.cnf </span> inside /mysql/etc
To help you we created one with some variables, please copy it
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo cp /workshop/support/my.cnf.mysql1 /mysql/etc/my.cnf </copy>
    ```
10. Check the content of the configuration file to have a look inside.
Please note that, because the port 3306 is already in use by the community server previously installed , we use now port 3307.
    ```
    <span style="color:green">shell-mysql1></span><copy> cat /mysql/etc/my.cnf </copy>
    ```
11. For security reasons change ownership and permissions
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo chown -R mysqluser:mysqlgrp /mysql </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo chmod -R 750 /mysql </copy>
    ```
  The following permission is for the Lab purpose so that opc account can make changes and copy files to overwrite the content
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo chmod -R 770 /mysql/etc </copy>
    ```
12. Save the changes, log out and log in again from the ssh for the changes to take effect on the user profile.initialize your database
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo /mysql/mysql-latest/bin/mysqld --defaults-file=/mysql/etc/my.cnf --initialize --user=mysqluser </copy>
    ```
13. Start your new mysql instance
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo /mysql/mysql-latest/bin/mysqld --defaults-file=/mysql/etc/my.cnf --user=mysqluser & </copy>
    ```
14. Verify that process is running
    ```
    <span style="color:green">shell-mysql1></span><copy>ps -ef | grep mysqld </copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>netstat -an | grep 3307 </copy>
    ```
15. Another way is searching the message “ready for connections” in error log as one of the last
    ```
    <span style="color:green">shell-mysql1></span><copy>grep -i ready /mysql/log/err_log.log </copy>
    ```
16. Retrieve root password for first login
    ```
    <span style="color:green">shell-mysql1></span><copy>grep -i 'temporary password' /mysql/log/err_log.log</copy>
    ```
17. Before version 5.7 it was recommended to run the ' mysql\_secure\_installation ' script. From version 5.7 all these settings are “by default”, but the script can be used also to setup the validate\_password plugin (used later).
    Execute now mysql\_secure\_installation
    ```
    <span style="color:green">shell-mysql1></span><copy>/mysql/mysql-latest/bin/mysql_secure_installation -h127.0.0.1 -P3307</copy>
    ```
    using these values
    * root password: retrieved from previous step
    * new password: Welcome1!
    * setup VALIDATE PASSWORD component: Y
    * password validation policy: 2 
    * Change the password for root: N
    * Remove anonymous users: Y
    * Disallow root login remotely: N
    * Remove test database: Y
    * Reload privilege tables now: Y

18. Login to you mysql-advanced installation and check the status.
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uroot -p -h 127.0.0.1 -P3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>status</copy>
    ```
19. Shutdown the service
    ```
    <span style="color:blue">mysql></span><copy>exit</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>mysqladmin -uroot -h127.0.0.1 -p -P3307 shutdown</copy>
    ```
20. Configure automatic startup and shutdown with system.
Add a systemd service unit configuration file with details about the MySQL service. The file is named mysqld.service and is placed in /usr/lib/systemd/system. We created one for you (See addendum for the content)
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo cp /workshop/support/mysqld-advanced.service /usr/lib/systemd/system/</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo chmod 644 /usr/lib/systemd/system/mysqld-advanced.service</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl enable mysqld-advanced.service</copy>
    ```

21. Test start, stop and restart
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl start mysqld-advanced</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl status mysqld-advanced</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl stop mysqld-advanced</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl status mysqld-advanced</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl restart mysqld-advanced</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo systemctl status mysqld-advanced</copy>
    ```
22. Create a new administrative user called 'admin' with remote access and full privileges
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uroot -p -h 127.0.0.1 -P3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```
23. In the configuration file was specified to load the commercial Thread Pool Plugin, check if it’s loaded and active. We use here the same command with different output (“;” vs “\G” as line termination)
    ```
    <span style="color:blue">mysql></span><copy>select * from information_schema.plugins where plugin_name like 'thread%';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>select * from information_schema.plugins where plugin_name like 'thread%'\G</copy>
    ```


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
