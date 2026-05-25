# Exercise 9: Restoring an AdventureWorks database backup taken from SQL Server 2012 instance [Read-only]
### Estimated Duration: 45 Minutes

## Lab Scenario

Contoso has some applications that use SQL Server as the backend database. They have installed SQL Server on their Windows servers in their manufacturing plants, but these locations don’t necessarily have local IT support to update the operating system and SQL Server with the latest security updates. They have explored Azure Database for SQL Server and found that it meets their requirements and offers some unique capabilities, such as being easy to manage and migrating from different cloud platforms. Therefore, they are excited about the opportunity of deploying SQL Server in their Azure Arc Arc-enabled environment.

## Objectives

In this exercise, you will be performing the following tasks:

- Task 1: Restore the AdventureWorks2012 database into Azure SQL Managed instance - Azure Arc Using Kubectl.
- Task 2: View Azure Arc Arc-enabled SQL-managed instance logs in Azure Portal.

## Task 1: Restore the AdventureWorks2012 database into Azure SQL Managed instance - Azure Arc Using Kubectl

Restoring an existing SQL database from a SQL Server to Azure Arc Arc-enabled SQL MI is very simple. All you have to do is take a backup from your existing SQL Server and then restore that backup to SQL MI. In this lab, we have already taken the backup and downloaded it to the local drive folder. 

Now let's copy and restore the already taken backup file into your Azure SQL Managed instance container using Kubectl commands.

1. Launch a **Command Prompt** window from the desktop of your JumpVM if you have already closed the existing one.

1. Run the following command to get the list of pods that are running on your data controller. 

   > **Note:** The namespace name for your data controller will be **azure-arc**.

   ```BASH
   kubectl get pods -n azure-arc
   ```
   
1. From the output of the above command, copy the pod name of the SQL MI instance from the output which will be in the following format sqlinstancename-0. If you followed the same naming convention as in the instructions, the pod name will be **arcsql-direct-0**.

   > **Note:** Please copy the Pod Name for the next step.

   ![](media/restore-direct-1.png "Confirm")
   
1. In the Command Prompt, run the following command after replacing the required values. This will remotely execute a command in the Azure SQL Managed instance container to copy the .bak file onto the container from the local directory.

   >**Note:** The value of the namespace name and pod name is already updated in the below command. Please confirm if the pod name that you had copied matches the one given below: arcsql-direct-0. 

   ```BASH
   cd C:\
   kubectl cp \AdventureWorks2012.bak arcsql-direct-0:var/opt/mssql/data/AdventureWorks2012.bak -n azure-arc
   ```

   ![](media/newcp.png "Confirm")

1. Now, to restore the AdventureWorks database, switch back to the Azure Data Studio and **right click on the Connection of your connected SQL Managed Instance Server (1)** and click on **New Query (2)**.

   ![](media/arc54.png "Confirm")

1. Once the query window is open, paste the below query **(1)** and click on **Run (2)** to execute it to restore the copied database to Azure Arc-enable SQL Managed instance 

   ```BASH
   RESTORE DATABASE AdventureWorks2012 FROM DISK = '/var/opt/mssql/data/AdventureWorks2012.bak'
   WITH MOVE 'AdventureWorks2012' to '/var/opt/mssql/data/AdventureWorks2012.mdf'  
   ,MOVE 'AdventureWorks2012_log' to '/var/opt/mssql/data/AdventureWorks2012_log.ldf'  
   ,RECOVERY;  
   GO
   ```

   ![](media/arc55.png "Confirm")

1. Then, right-click on the **arcsql-direct (1)** SQL Managed Instance Server under the CONNECTIONS tab on the top left of the Azure Data Studio and click on **Refresh (2)**.

   ![](media/arc56.png "Confirm")

1. Now expand your SQL Managed Instance server if not already by clicking on the arrow icon on the left of the IP Address, then expand **Databases** and verify that the **AdventureWorks2012** database is listed there.

   ![](media/arc57.png "Confirm")

## Task 2: View Azure Arc Arc-enabled SQL-managed instance logs in Azure Portal

In this task, you will use the Azure Portal to access your Log Analytics workspace. You’ll explore the sqlManagedInstances_agent_logs_CL table under Custom Logs to view diagnostic and operational logs from your Arc-enabled SQL Managed Instance, using the KQL query editor.

1. Navigate to [Azure Portal](https://portal.azure.com/#home) and then search for **Log Analytics workspace** in the search bar at the top and then select it.

   ![](./media/search-law.png "Lab Environment")

1. In the **Log Analytics workspaces** page, select **LoganalyticsWS-Direct** workspace.
   
    ![](media/hybrid65.png "Confirm")

1. Then, from the left navigation menu select **Logs** **(1)** and on the Queries tab, click on the ```X``` **(2)** at the top right corner as shown in the below image.

   ![](media/hybrid71.png "Confirm")
   
1. And then, click on ```>>``` icon to expand the Schema and Filter tab.

    ![](media/logaw-2.png "Confirm")

1. Check for CustomLogs under the Tables section. If you do not see CustomLogs under Tables, refresh the page every 2 minutes until it is available.
     
    ![](media/logaw-3.png "Confirm")

1. Once the Custom logs are available, expand Custom Logs **(1)** at the bottom of the list of tables and you will see a table called **sqlManagedInstances_agent_logs_CL (2)** select it.
   
    ![](media/hybrid79.png "Confirm")

1. Set the mode to **KQL mode** 

    ![](media/arc59.png "Confirm")

1. Now, you will have a query in the query editor. Run the query that will show the logs by clicking on **Run** **(1)** button and explore the **Results** **(2)**. 
   
    ![](media/logaw-6.png "Confirm")

    > Note: You might have to resize the editor to view the logs from the output window.

## Summary

In this exercise, you restored the AdventureWorks database into an Azure Arc-enabled SQL Managed Instance, viewed SQL instance logs in the Azure portal.

### You have successfully completed the lab.

By completing this lab, you gained hands-on experience with Azure Arc to extend Azure management, security, and data services to hybrid and multicloud environments. You onboarded servers and SQL instances to Arc for centralized monitoring and governance, integrated Microsoft Defender for Cloud and Microsoft Sentinel for enhanced security, and enabled Azure Automanage for streamlined VM lifecycle management. You also implemented GitOps configurations on Azure Arc-enabled Kubernetes clusters, enforced compliance using Azure Policy, and configured Azure Monitor for container insights. Finally, you deployed Azure Arc Data Controllers and SQL Managed Instance in direct connectivity mode, restored databases, and validated management capabilities using Azure Data Studio. This end-to-end exercise demonstrated how to build, secure, and operate a hybrid cloud solution consistently across on-premises, edge, and multicloud infrastructures.