# MySQL Enterprise Monitor

## Introduction
This addendum contains additional information and troubleshooting guides. 


## Task: Content of file my.cnf mysql1

 ![MYSQLEE](images/content-my-cnf-mysql1.png "content my cnf mysql1")

## Task: Content of file my.cnf mysql2

  ![MYSQLEE](images/content-my-cnf-mysql2.png "content my cnf mysql2")

## Task: Content of file my.cnf mysql3

  ![MYSQLEE](images/content-my-cnf-mysql3.png "content my cnf mysql3")

## Task: Content of mysqld-advanced

 ![MYSQLEE](images/content-mysqld-advanced.png "content mysqld advanced")

## Task: Content of file mysqlbackup\_user\_grants

 ![MYSQLEE](images/content-mysqld-backup-user-grants.png "content mysqld backup user grants")

## Task: Putty configuration 
 Steps to configure putty connection to OCI compute instances.
1. Open putty
2. Insert the public IP of your server and a mnemonic session name
    ![MYSQLEE](images/putty-config.png "putty config")
3. Choose "Connection→SSH→Auth" and provide the id_rsa.ppk path
    ![MYSQLEE](images/putty-auth.png "putty auth")

4. Select "Connection→Data" and insert "opc" in "Auto-login username"
    ![MYSQLEE](images/putty-data.png "putty-data")

5. Choose Connection and insert "60" in "Seconds between keepalives"
    ![MYSQLEE](images/putty-connection.png "putty connection")

6. Return to "Session" and click save
    ![MYSQLEE](images/putty-save.png "putty save")


## Task: How to create and use Compartments 
> **Note:**
 * Two Compartments, named Oracle Account Name (root) and a compartment for PaaS, were
automatically created by the Oracle Cloud
 * For more information on Managing Compartments, please visit:
https://docs.cloud.oracle.com/iaas/Content/Identity/Tasks/managingcompartments.htm.

1. Login in your OCI tenancy dashboard
2. On the Navigation Menu, under “Identity and Security”, select Identity -> Compartments.
    ![MYSQLEE](images/oci-landing-page.png "oci landing page")

3. On Compartments Page, click on “Create Compartment”.
    ![MYSQLEE](images/oci-create-compartment.png "oci create compartment")

4. On Create Compartment, enter Name, Description, select Parent Compartment, and click on Create Compartment.
    ![MYSQLEE](images/oci-compartment-config.png "oci compartment config")

5. Completed Compartments
    ![MYSQLEE](images/oci-compartments.png "oci compartments")

6. Now to work in your compartment, choose it from left side menu when you create services. Here a screenshot from “Compute/Instances”
    ![MYSQLEE](images/oci-compartment-choose.png "oci compartment choose")

## Task: Troubleshooting: Windows Powershell SSH error on permissions 
1. In case you are using powershell and you receive an error like
    
    ![MYSQLEE](images/powershell-permissions-error.png "powershell permissions error")

    Please use below commands to fix the permissions (please change full path with your path (e.g. New-Variable -Name Key -Value "C:\mysql\id_rsa"))

    ```
    <span style="color:green">powershell></span><copy>New-Variable -Name Key -Value "<full path>\id_rsa"</copy>
    ```

    Verify that ls command retrieve the right file:
    ```
    <span style="color:green">powershell></span><copy>ls $Key</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>Icacls $Key /c /t /Inheritance:d</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>Icacls $Key /c /t /Grant ${env:UserName}:F</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>TakeOwn /F $Key</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>Icacls $Key /c /t /Grant:r ${env:UserName}:F</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>Icacls $Key /c /t /Remove:g Administrator "Authenticated Users" BUILTIN\Administrators BUILTIN Everyone System Users</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>Icacls $Key</copy>
    ```
    ```
    <span style="color:green">powershell></span><copy>Remove-Variable -Name Key</copy>
    ```

    Then retry the ssh connection.

## Task: Troubleshooting: Lost private key for app-srv 
 If the private ssh keys to access your app-srv server is lost (because inside a broken laptop), the recovey is really time consuming and difficult.
 The best approach is to create a custom image of your server and use it to recreate the server.
1. Connect to your tenancy
2. Select “instances” from “Compute”
    ![MYSQLEE](images/oci-menu-page.png "oci menu page")

3. Select the app-srv server and show details
4. Take a note of the <span style="color:red">actual private IP address</span>!
    * If you forgot this part, you have to manually update the /etc/fstab file on each mysqlX instance, then remount the NFS share
    ![MYSQLEE](images/oci-instance-details.png "oci instance details")

5. From the menu “More Actions” select “Create custom image”
    ![MYSQLEE](images/oci-custom-image.png "oci custom image")

6. Wait that the image is created
7. Select “custom images” and check that your image is available
    ![MYSQLEE](images/oci-custom-image-select.png "oci custom image select")
8. <span style="color:red">If the image is not Available, not read the error and fix it before continuing</span>
9. Return to “Instances” and... have you saved the private IP?
    If not, save it now and then delete (“Terminate”) the existing app-srv
    ![MYSQLEE](images/oci-terminate-instance.png "oci terminate instance")
    ![MYSQLEE](images/oci-terminate-instance-confirm.png "oci terminate instance confirm")

10. Go to “Custom images” and select your image
    ![MYSQLEE](images/oci-custom-images-page.png "oci custom images page")
11. Deploy a new app-srv server with “Create Instance” button.
    * Use app-srv as name
    * Expand “Show Advanced Option” in network section and insert the previous private IP address
    * Create a new private key (<span style="color:red">SAVE IT !!!</span>)
    ![MYSQLEE](images/oci-create-instance.png "oci create instance")
    ![MYSQLEE](images/oci-create-instance-details.png "oci create instance details")
12. When the instance is available, reconnect with the <span style="color:red">NEW PUBLIC IP</span> and the <span style="color:red">NEW PRIVATE KEY!</span>
13. The content is the same as the lost server



## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023