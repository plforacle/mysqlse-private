# User management

## Introduction
In this lab you will explore user creation, their privileges and roles.

Estimated Lab Time: 20 minutes

### Objectives
In this lab, you will:
* Explore user creation and privileges 
* Discover and Configure roles


## Task 1: User creation and privileges
> **Note:** 
  * Servers:
    * app-srv as client (used by appuser)
    * mysql1 as mysql server
  * Use two shells, both can be from app-srv

1. If not already connected, connect to app-srv and retrieve the Private IP (this info is also available in OCI dashboard)
    
    Write it down.
    ```
    <span style="color:green">shell-app-srv$</span><copy>ifconfig | grep 10.0</copy>
    ```
    
2. <span style="color:red">Keep previous connection open and open a new connection </span>to mysql1 server thought app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```
3. Connect to your <span style="color:red">mysql-advanced</span> with administrative user (<span style="color:red">keep it open</span>)
    ```
    <span style="color:green">shell-app-srv$</span><copy>mysql -uadmin -p -h mysql1 -P 3307</copy>
    ```
    * Create a new user and restrict the user to your “app-srv” IP
        ```
        <span style="color:blue">mysql></span><copy>CREATE USER 'appuser'@<app-srv_ip> IDENTIFIED BY 'Welcome1!';</copy>
        ```
        ```
        <span style="color:blue">mysql></span><copy>GRANT ALL PRIVILEGES ON world.* TO 'appuser'@<app-srv_ip>;</copy>
        ```
        ```
        <span style="color:blue">mysql></span><copy>SHOW GRANTS FOR 'appuser'@<app-srv_ip>;</copy>
        ```
4. Open a <span style="color:red">new shell on app-srv</span> install the clients to test appuser connection
    * Install mysql client and mysqlsh
        ```
        <span style="color:green">shell-app-srv$</span><copy>sudo yum -y install /workshop/linux/client/*.rpm</copy>
        ```
    * Connect to mysql-advanced with appuser
        ```
        <span style="color:green">shell-app-srv$</span><copy>mysql -u appuser -p -h mysql1 -P 3307</copy>
        ```
    * Run a select on the tables e.g.
        ```
        <span style="color:blue">mysql></span><copy>USE world;</copy>
        ```
        ```
        <span style="color:blue">mysql></span><copy>SELECT * FROM city;</copy>
        ```
5. (<span style="color:red">admin connection</span>) Switch to the administrative connection revoke privilege on city to appuser
    ```
    <span style="color:blue">mysql></span><copy>REVOKE SELECT ON world.* FROM 'appuser'@<app-srv_ip>;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW GRANTS FOR 'appuser'@<app-srv_ip>;</copy>
    ```
6. (<span style="color:red">appuser connection</span>) Repeat the select on for the user. There is a difference?
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM city;</copy>
    ```
7. (<span style="color:red">appuser connection</span>) Close and reopen the appuser connection, then repeat above commands. There is a difference?
    ```
    <span style="color:blue">mysql></span><copy>exit</copy>
    ```
    ```
    <span style="color:green">shell-app-srv$</span><copy>mysql -u appuser -p -h mysql1 -P 3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>USE world;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM city;</copy>
    ```
8. (<span style="color:red">admin connection</span>) Switch to the administrative connection revoke ‘USAGE’ privilege using and administrative connection and verify 
    (tip: this privilege can’t be revoked...)
    ```
    <span style="color:blue">mysql></span><copy>REVOKE USAGE ON *.* FROM 'appuser'@<app-srv_ip>;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW GRANTS FOR 'appuser'@<app-srv_ip>;</copy>
    ```
9. (<span style="color:red">admin connection</span>) Using the administrative connection revoke all privileges using and administrative connection and verify
    ```
    <span style="color:blue">mysql></span><copy>REVOKE ALL PRIVILEGES ON *.* FROM 'appuser'@<app-srv_ip>;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW GRANTS FOR 'appuser'@<app-srv_ip>;</copy>
    ```
10. (<span style="color:red">appuser connection</span>) Close and reopen appuser session, do you see schemas?
    ```
    <span style="color:blue">mysql></span><copy>exit</copy>
    ```
    ```
    <span style="color:green">shell-app-srv$</span><copy>mysql -u appuser -p -h mysql1 -P 3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
11. (<span style="color:red">admin connection</span>) Using the administrative connection restore user privileges to reuse it in next labs
    ```
    <span style="color:blue">mysql></span><copy>GRANT ALL PRIVILEGES ON world.* TO 'appuser'@<app-srv_ip>;</copy>
    ```
12. (<span style="color:red">admin connection</span>) Using the administrative connection, what are your password settings?
    ```
    <span style="color:blue">mysql></span><copy>SHOW VARIABLES LIKE 'validate_password%';</copy>
    ```
13. (<span style="color:red">appuser connection</span>) Try to set unsecure passwords for appuser
    ```
    <span style="color:blue">mysql></span><copy>alter user 'appuser'@<app-srv_ip> identified by 'appuser';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>alter user 'appuser'@<app-srv_ip> identified by 'Welcome';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>alter user 'appuser'@<app-srv_ip>identified by 'We1!';</copy>
    ```

14. (<span style="color:red">admin connection</span>) Expire the password for appuser
    ```
    <span style="color:blue">mysql></span><copy>alter user 'appuser'@<app-srv_ip> PASSWORD EXPIRE;</copy>
    ```
15. (<span style="color:red">appuser connection</span>) Close and reopen connection to mysql-advanced and try to submit a command.
    What changed?
    ```
    <span style="color:green">shell-app-srv$</span><copy>mysql -uappuser -p -h mysql1 -P 3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>alter user 'appuser'@<app-srv_ip> identified by 'Welcome1!';</copy>
    ```

## Task 2: User Roles
 > **Note:**
  * pp-srv can be used for this task
  * mysql1 is the MySQL Server

1. If not already connected, connect to mysql1 through app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```
2. Reconnect to MySQL instance as <span style="color:red">admin</span> to create a new user
    ```
    <span style="color:green">shell></span><copy>mysql -u admin -p -h mysql1 -P 3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE USER 'appuser2'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW GRANTS FOR 'appuser2'@'%';</copy>
    ```
3. Now create a new role and grant it to the new user
    ```
    <span style="color:blue">mysql></span><copy>CREATE ROLE 'app_read';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>GRANT SELECT ON world.* TO 'app_read';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>GRANT 'app_read' TO 'appuser2'@'%';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW GRANTS FOR 'appuser2'@'%';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>exit</copy>
    ```
4. Connect now as <span style="color:red">appuser2</span> and submit some commands (you receive errors, why?)
    ```
    <span style="color:green">shell></span><copy>mysql -u appuser2 -p -h mysql1 -P 3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT current_role();</copy>
    ```
5. Set the role for the user and repeat above commands
    ```
    <span style="color:blue">mysql></span><copy>SET ROLE ALL;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT current_role();</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT COUNT(*) FROM world.city;</copy>
    ```
6. We can also assign a default role to don’t ask to set one after every login
    ```
    <span style="color:blue">mysql></span><copy>ALTER USER 'appuser2'@'%' DEFAULT ROLE 'app_read';</copy>
    ```



## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
