# MySQL Enterprise Masking and De-identification

## Introduction
MySQL Enterprise Masking and De-identification provides an easy to use, built-in database solution to help organizations protect sensitive data from unauthorized uses by hiding and replacing real values with substitutes.

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Install and use data masking functionalities


> **Note:**
 * Server: mysql1
 * The lab can be completed from app-srv
 * Data masking has more functions than what we test in the lab. Find the full list of functions in the Learn More section of this lab.

## Task 1: Enable and test Datamasking and Deidentification

1. Install the data masking plugin
    ```
    <span style="color:green">shell></span><copy>mysql -uadmin -p -h mysql1 -P 3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>INSTALL PLUGIN data_masking SONAME 'data_masking.so';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SHOW PLUGINS;</copy>
    ```
2. Look for data_masking and check the status? Is it active?
3. Install some masking functions
    ```
    <span style="color:blue">mysql></span><copy>CREATE FUNCTION gen_range RETURNS INTEGER SONAME 'data_masking.so';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE FUNCTION gen_rnd_email RETURNS STRING SONAME 'data_masking.so';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE FUNCTION gen_rnd_us_phone RETURNS STRING SONAME 'data_masking.so';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE FUNCTION mask_inner RETURNS STRING SONAME 'data_masking.so';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>CREATE FUNCTION mask_outer RETURNS STRING SONAME 'data_masking.so';</copy>
    ```

4. Use data masking functions
    ```
    <span style="color:blue">mysql></span><copy>SELECT mask_inner(NAME, 1,1) FROM world.city limit 10;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT mask_outer(NAME, 1,1) FROM world.city limit 10;</copy>
    ```
5. Discuss differences between mask\_inner and mask\_outer
    ```
    <span style="color:blue">mysql></span><copy>SELECT mask_inner(NAME, 1,1, '&') FROM world.city limit 1;</copy>
    ```
6. Use data masking random generators executing these statements several times
    ```
    <span style="color:blue">mysql></span><copy>SELECT gen_range(1, 200);</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT gen_rnd_us_phone();</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT gen_rnd_email();</copy>
    ```


## Learn More
* https://dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-usage.html 


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
