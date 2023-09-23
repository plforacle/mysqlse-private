# MySQL Shell: the new client

## Introduction
In this lab you will download and start using MySQL Shell.

Estimated Lab Time: 20 minutes

### Objectives
In this lab, you will execute MySQL Shell and explore the following on the interface:
* Help
* Settings
* Test an extension: reporting
* Command mode

> **Note:** 
  * Server: mysql1
  * To close the MySQL Shell you can the commands “\q” or “\exit”

## Task 1: Download MySQL Shell

1. If not already connected, connect to mysql1 server through app-srv
    ```
    <span style="color:green">shell-app-srv$</span><copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```
2. Install Mysql Shell, a new client that can be used in devOps organizations (you’ll learn more about it during the course) 
    ```
    <span style="color:green">shell-mysql1></span><copy>sudo yum install /workshop/linux/mysql-shell-commercial-8.0.*.rpm</copy>
    ```
3. Launch MySQL shell
    ```
    <span style="color:green">shell-mysql1></span><copy>mysqlsh</copy>
    ```
4. Discover the command auto-completion feature, type \h and press TAB twice
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\h</copy> [TAB][TAB]
    ```
5. Check the available options. Add the letter “e” to “\h” and press TAB again to see that the command will automatically complete for you. Press enter and explore the help menu
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\he</copy> [TAB]
    ```
6. Activate the command history autosave in MySQL shell
    MySQL Shell comes with the option of automatically saving the history of command disabled by default. Therefore we need to check and to activate it.
    * Show settings and look for history.autoSave
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\option -l</copy>
    ```
    * Activate autosave history
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\option --persist history.autoSave=1</copy>
    ```
7. Close and reopen the session and in the new one **uses the arrow up key** to verify that the data from previous session are available
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>exit</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>mysqlsh</copy>
    ```

## Task 2: MySQL as a SQL Client

1. MySQL Shell can be used as SQL client
    * Connect to the newly installed mysql-advanced instance. Enter the password when requested and press enter. 
    
        When the prompt asks to save the password choose No and press Enter.
        ```
        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\c admin@mysql1:3307</copy>
        ```
    * Switch to SQL mode
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\sql</copy>
    ```
2. Now you can submit SQL commands
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>show databases;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>SELECT * FROM world.city LIMIT 10;</copy>
    ```
3. MySQL Shell has also nice features. Here we see one of them: reporting.
    If you want to execute a command continuously use the command “query” (default refresh time: 5 seconds).
    To exit from reporting press CTRL+C
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\watch query show processlist;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\watch query --interval=2 show processlist;</copy>
    ```
4. Switch to Python command mode, then Javascript command mode, check how prompt change
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\py</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:blue">Py</span></span>><copy>\py</copy>
    ```
5. Exit from MySQL Shell with “\q” or “\exit”
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\q</copy>
    ```
6. Command line connection from MySQL Client and MySQL Shell are similar (just specify “--sql”). Try these:
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -uadmin -p -hmysql1 -P3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>exit</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span><copy>mysqlsh --sql -uadmin -p -hmysql1 -P3307</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\exit</copy>
    ```
7. From now on, if you like, the labs may be completed with MySQL Shell instead of classic mysql client.


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
