# Exercise 4: Enabling Azure Automanage for Server - Azure Arc [Read-only]

### Estimated Duration: 30 Minutes

## Lab Scenario

Your organization is standardizing the management of hybrid servers by adopting automated governance and operational best practices. With multiple servers connected through Azure Arc, manually configuring security, monitoring, and updates can be time-consuming and error-prone.

In this exercise, you will enable Azure Automanage on an Azure Arc-enabled server to automatically apply recommended configurations for security, compliance, monitoring, and updates. By using the predefined Dev/Test configuration profile, you will simplify server management and ensure consistency across environments. By the end of this exercise, the server will be continuously monitored and maintained according to Azure best practices with minimal manual intervention.

## Objective

In this exercise, you will be performing the following task:

- Task 1: Configuring Azure Automanage

## Task 1: Configuring Azure Automanage

In this task, you will enable Azure Automanage on the sqlvm Arc-enabled server. Automanage applies best practices for VM management, such as security, updates, and monitoring, using services like Azure Security Center and Log Analytics. You will use the Dev/Test configuration profile to automate and simplify the VM’s management lifecycle.

1. Navigate to the home page of the [Azure Portal](https://portal.azure.com/#home), then search for **Automanage (1)** in the search box and select **Automanage (2)**.

   ![](.././media/arc27.png "searchautoamanage")
   
2. From the Automanage pane, select **Automanage machines (1)** under Machine best practices and click on **+ Enable on existing machine (2)**.

   ![](.././media/hybrid18.png "searchautoamanage")

3. On Enable Automanage - Azure machine best practices page, select **Azure best practices - Dev / Test (1)** and click on **Next: Machines (2)**.

   ![](.././media/hybrid45.png "searchautoamanage")

4. In Select machines pane, select the **sqlvm (1)** server and click on **Review + Create (2)**.

   ![](.././media/new/t1.png)

5. Click on **Create**.

   ![](.././media/new/t2.png)

6. Once the Configuration profile assignment is completed successfully, it will take around `20-30 minutes` to get the Status as **Conformant**.

   ![](.././media/new/t3.png)

7. You can proceed with the next task and review the status later.

## Summary

In this task, you used Azure Automanage to enroll and configure a VM, applying the "Azure best practices - Dev/Test" profile. This setup automates lifecycle management, including security, updates, change tracking, and monitoring through tools like Azure Security Centre and Log Analytics.

### You have successfully completed the exercise. Click on **Next >>** from the bottom right corner to proceed with the next exercise.

 ![](.././media/arcg6.png)