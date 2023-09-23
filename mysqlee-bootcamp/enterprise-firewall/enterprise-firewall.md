# MySQL Enterprise Firewall

## Introduction
MySQL Enterprise Firewall guards against cyber security threats by providing real-time protection against database specific attacks. It rotects the data by monitoring, alerting, and blocking unauthorized database activity without any changes to applications.

In this lab you will work with group profiles, which are used to create a composite list of allowed queries for a group of users and enforces firewall protection across all in the group profile.
You will also work with account profiles, as they are also used in MySQL 5.6 and 5.7 and have the same approach.

Estimated Lab Time: 30 minutes

### Objectives
In this lab, you will work with:
* Account Profiles
* Group Profiles


> **Note:**
 * Server: mysql1

## Task 1: Account Profiles

1. If not already connected, connect to mysql1 through app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```
2. Install MySQL Enterprise Firewall on mysql-advanced using CLI
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uadmin -p -P3307 -h mysql1 < /mysql/mysql-latest/share/linux_install_firewall.sql</copy>
    ```
3. Connect to the instance with administrative account <span style="color:red">first SSH connection - administrative</span>
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uadmin -p -P3307 -hmysql1</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL VARIABLES LIKE 'mysql_firewall_mode';</copy>
    ``````
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL STATUS LIKE "firewall%";</copy>
    ```
4. Create a new user 'fwtest' and assign full privileges to database world
    ```
    <span style="color:blue">mysql></span><copy>CREATE USER 'fwtest'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>GRANT ALL PRIVILEGES ON world.* TO 'fwtest'@'%';</copy>
    ```
5. Now we set firewall in recording mode, to create a white list and verify that allowlist is empty
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_mode('fwtest@%', 'RECORDING');</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM mysql.firewall_users;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM mysql.firewall_whitelist;</copy>
    ```
6. Open a <span style="color:red">second SSH connection</span> and use it to connect as “fwtest” from app-srv
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -ufwtest -p -P3307 -hmysql1</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>USE world;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM city limit 25;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 200000;</copy>
    ```
7. <span style="color:red">Administrative connection:</span> Return to admin session (first SSH connection terminal) and verify that there are now rules in allowlist (noticed that we interrogate temporary rules from information schema)
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM information_schema.mysql_firewall_whitelist;</copy>
    ```
8. <span style="color:red">Administrative connection:</span> switch Firewall mode from 'recording' to 'protecting' and verify the presence of rules in allowlist
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_mode('fwtest@%', 'PROTECTING');</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM mysql.firewall_whitelist;</copy>
    ```
9. <span style="color:red">fwtest connection:</span> run these commands. Which one’s work? Which ones fail and why?
    ```
    <span style="color:blue">mysql></span><copy>USE world;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM city limit 25;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 200000;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM countrylanguage;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 500000;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 200000 or 1=1;</copy>
    ```
10. <span style="color:red">Administrative connection:</span> set firewall in detecting mode in your
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_mode('fwtest@%', 'DETECTING');</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SET GLOBAL log_error_verbosity=3;</copy>
    ```
11. <span style="color:red">fwtest connection:</span> Repeat a blocked command (it works? Why?)
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM world.country WHERE population > 200000 or 1=1;</copy>
    ```
12. Search the error in the error log.
    ```
    <span style="color:green">shell-mysql1></span><copy>grep "MY-011191" /mysql/log/err_log.log</copy>
    ```
13. <span style="color:red">Administrative connection:</span> Error log can also be interrogated also from the client.
    ```
    <span style="color:blue">mysql></span><copy>select * from performance_schema.error_log order by LOGGED DESC limit 5;</copy>
    ```
14. Disable now the firewall
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_mode('fwtest@%', 'OFF');</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT MODE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS WHERE USERHOST = 'fwtest@%';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT RULE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_WHITELIST WHERE USERHOST = 'fwtest@%';</copy>
    ```
 
## Task 2: Group Profiles
1. If not already connected, connect to mysql1 through app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```
2. Connect to the instance with administrative account and create a new user 'fwtest2', then assign full privileges to database world
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uadmin -p -P3307 -hmysql1</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE USER 'fwtest2'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>GRANT ALL PRIVILEGES ON world.* TO 'fwtest2'@'%';</copy>
    ```
3. Grant firewall management privilege to admin and exclude it from firewall checks
    ```
    <span style="color:blue">mysql></span><copy>GRANT FIREWALL_ADMIN, FIREWALL_EXEMPT on *.* to admin@'%';</copy>
    ```
4. Now create a firewall group profile in recording mode, and add the new user
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_group_mode('fwgrp', 'RECORDING');</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_firewall_group_enlist('fwgrp', 'fwtest2@%');</copy>
    ```
5. Open a <span style="color:red">second SSH</span> connection and use it to connect as “fwtest2” from app-srv
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -ufwtest2 -p -P3307 -hmysql1</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>USE world;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM city limit 25;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 200000;</copy>
    ```
6. <span style="color:red">Administrative connection:</span> Return to admin session (first SSH connection terminal) and verify that there are now rules in allowlist
    ```
    <span style="color:blue">mysql></span><copy>SELECT RULE FROM performance_schema.firewall_group_allowlist WHERE NAME = 'fwgrp';</copy>
    ```
7. <span style="color:red">Administrative connection:</span> switch Firewall mode from 'recording' to 'protecting' and verify the presence of rules in allowlist
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_group_mode('fwgrp', 'PROTECTING');</copy>
    ```
8. <span style="color:red">Fwtest2 connection:</span> run these commands. Which one’s work? Which ones fail and why?
    ```
    <span style="color:blue">mysql></span><copy>USE world;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM city limit 25;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 200000;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM countrylanguage;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 500000;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT Code, Name, Region FROM country WHERE population > 200000 or 1=1;</copy>
    ```
9. Disable now the firewall
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_group_mode('fwgrp', 'OFF');</copy>
    ```



## Learn More
* https://dev.mysql.com/doc/refman/8.0/en/firewall-installation.html


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
