# MySQL Enterprise Audit

## Introduction
MySQL Enterprise Audit provides an easy to use, policy-based auditing solution that helps implement stronger security controls and satisfy regulatory compliance.

Audit can be activated and configured without stop the instance. In this lab we edit my.cnf and restart the instance just to see how to update my.cnf

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will :
* See how MySQL Enterprise Audit works


> **Note:**
 * Server: mysql1

## Task 1: Enable and test Audit

1. If not already connected, connect to mysql1 through app-srv
2. Enable Audit Log on mysql-advanced (remember: you can’t install on mysql-gpl). Audit is COMMERCIAL plugin.
    * Edit the my.cnf setting in /mysql/etc/my.cnf
        ```
        <span style="color:green">shell-mysql1></span><copy>sudo nano /mysql/etc/my.cnf</copy>
        ```
    * Change the line “plugin-load=thread_pool.so” to load the plugin
        ```
        plugin-load=thread_pool.so;<copy>audit_log.so</copy>
        ```
    * <span style="color:red">below the previous</span> add these lines to make sure that the audit plugin can't be unloaded and that the file is automatically rotated at 20 MB
        ```
        <copy>audit_log=FORCE_PLUS_PERMANENT</copy>
        <copy>audit_log_rotate_on_size=20971520</copy>
        ```
    * Restart MySQL (you can configure audit without restart the server, but here we show how to set the configuration file)
        ```
        <span style="color:green">shell-mysql1></span><copy>sudo systemctl restart mysqld-advanced</copy>
        ```

3. Login to a mysql-advanced with the user “appuser”, then submit some commands
    ```
    <span style="color:green">shell-mysql1></span><copy>mysql -u appuser2 -p -h mysql1 -P 3307</copy>
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
4. Open the audit.log file the datadir and verify the content
    ```
    <span style="color:green">shell-mysql1></span><copy>nano /mysql/data/audit.log</copy>
    ```
5. You can check the documentation about other Log policies


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
