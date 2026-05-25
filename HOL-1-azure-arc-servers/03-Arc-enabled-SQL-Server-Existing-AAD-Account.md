# Exercise 3: Onboard SQL Server to Arc
### Estimated Duration: 45 Minutes
## Overview 

## Lab Scenario

Your organization is expanding its hybrid cloud strategy by bringing critical data services under centralized management using Azure Arc. As part of this initiative, SQL Server instances running in non-Azure environments need to be onboarded to Azure to enable unified governance, monitoring, and security.

In this exercise, you will onboard an existing SQL Server instance to Azure Arc using both the Azure Portal and PowerShell. You will then configure monitoring by integrating it with Azure services and run an on-demand SQL Best Practices Assessment. This will help identify configuration issues, security gaps, and optimization opportunities. By the end of this exercise, you will have full visibility and management capabilities for your SQL Server through Azure Arc.

## Objectives

In this exercise, you will be performing the following tasks:

- Task 1: Log in To Azure Portal
- Task 2: Register Azure Arc-enabled SQL Server.
- Task 3: Run on-demand SQL Assessment.

## Task 1: Log in To Azure Portal

In this task, you will begin by navigating the Azure Portal to onboard an existing SQL Server instance to Azure Arc. You’ll use the graphical interface to initiate the onboarding wizard, provide resource group and region details, and generate a PowerShell script that facilitates the registration process. This script will be executed in the next task to connect the SQL Server instance to Azure Arc. This step ensures that SQL Server is discoverable and manageable from Azure.

1. From the **Azure Portal**, search for **Azure Arc (1)** in the search box and select **Azure Arc (2)** from the services. 

    ![](.././media/new/a6.png)
   
1. From the left navigation pane, expand **Data services (1)**, select **SQL servers (2)** and under **SQL Server instances**, click on **+ Add (3)**.
 
   ![](.././media/new/e1.png)
   
1. In the Adding existing SQL Server instances page, click on **Connect SQL Server instances**.

   ![](.././media/arc-75.png "sqlsearch")
   
1. You will now see the prerequisite page. You can explore the page and then click on the **Next: Server details** option.
    
   > **Note:** We have already completed the prerequisite part for you. 
    
   ![](.././media/presql.png "sqlsearch")
   
1. On the **Server Details** blade, enter the below details.
 
   - **Subscription:** Leave default **(1)**

   - **Resource group:** Select **azure-arc (2)** from the dropdown list.

   - **Region:** Select the same region as the Resource group. **(3)**

   - **Operating systems:** Select **Windows (4)**.

   - **Server Name:** Type **sqlvm (5)**

   - **License Type:** Select **I have a production environment on this server with Enterprise or Standard edition covered by Software Assurance or SQL subscription ("Paid") (6)**.

   - Now, click on the **Next: Tags (7)** button.
   
      ![](.././media/new/e2.png)
   
1. Leave the default for tags blade and click on **Next: Run Script** button.
 
1. On the **Run Script** blade, explore the given script. We will be using this PowerShell script to **Register Azure Arc enabled SQL Server** later.
 
   > **Note:** Please **skip the script download** from here by clicking on ``X`` at the top right as we have **already downloaded** this script inside the Lab VM for you.
    
   ![](.././media/new/e3.png)
     
## Task 2: Register Azure Arc-enabled SQL Server.

Now that the registration script is prepared, you will switch to the LabVM/ARCHost VM and execute a PowerShell script that registers the SQL Server instance to Azure Arc. This task links the on-prem SQL Server running on the sqlvm virtual machine to Azure, enabling visibility and management from the Azure Portal. If the initial registration doesn't show a "Connected" state, you’ll also learn how to manually re-register the SQL Server using service principal credentials and remote PowerShell execution.

1. Minimize the Azure Portal Browser window. 

1. From the desktop of your **LabVM/ARCHost VM**, double click on **Windows PowerShell** icon to open it.
 
   ![](.././media/powershell.png "sqlsearch")
  
1. Then, run the below command to change the directory to where the script gets downloaded.
 
   ``` 
   cd C:\LabFiles
   ```

1. After changing the directory to **Lab files**, run the command given below:

   ```
   .\Execute-RegisterSqlServerArc.ps1
   ```
     
   > **Note** : This will initiate the execution of **RegisterSqlServerArc.ps1** script inside **sqlvm** that is deployed on Hyper-V.

   > **Note:** Make sure that the **sqlvm** is in running state on **Hyper-V**.

1. While the script is executing, poweshell will ask you for device-authentication. Copy the **code** provided from powershell.

   ![](.././media/run.png "sqlsearch")

1. Open https://microsoft.com/devicelogin in a brower and **paste the code (1)** and click on **Next (2)** to select the azure account to authenticate.

   ![](.././media/newdevlogin.png "newimage")

1. Click on **Continue** to confirm your login.

   ![](.././media/loginconfirm.png "loginconfirm")
  
1. After some time, you will see that the script execution is completed. Make sure that you see the output as shown in the image below.

   ![](.././media/completed.png "sqlsearch")

   > **Note:** Please wait until the commands finish executing. If the script appears to be stuck, press **Enter** a few times to continue.

1. Bring back the browser window where you had opened Azure Portal and search for **Azure Arc | SQL Server instances**. If you are already on that page, you will need to click on the Refresh button. On that page, you will see one resource **SQLVM** that we just created using the PowerShell script in the previous step.

   ![](.././media/new/e4.png)

   > **NOTE:** Wait for 5-10 minutes to show from registered to connected, if you don't see the Mode as **Connected**, then open a new Powershell window and run the below script, ensure to update the values in the `$block` section to define your variables. You can fetch these values from the **Environment > Service Principal Details** tab.

   ```
   $block = {
      # Import Environment Credentials
      CD C:\LabFiles
      $AppID = "<Application ID>"
      $AppSecret = "<Secret Key>"
      $TenantID = "<Tenant ID (Directory ID)>"  
      $SubscriptionId = "<Subscription ID>"
      $ResourseGroup = "azure-arc"
      $location = "<Resource group Region>"

      # Convert secret key to a secure string
      $passwd = ConvertTo-SecureString -AsPlainText -Force -String $AppSecret
      $pscredential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $AppID, $passwd

      # Login to Azure using Service Principal
      Connect-AzAccount -ServicePrincipal -Credential $pscredential -Tenant $TenantID

      # Set Execution Policy to allow script execution
      Set-ExecutionPolicy Bypass -Scope Process -Force

      # Run RegisterSqlServerArc.ps1 script to register SQL VM Server and SQL Server on Azure Arc
      & '.\RegisterSqlServerArc.ps1'
   }

   # Administrator password (should be securely handled)
   $ap = "demo@pass123"

   # Create credential object for remote authentication (Fixed -TypeName placement)
   $securePassword = ConvertTo-SecureString -AsPlainText -Force -String $ap
   $cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "Administrator", $securePassword

   # Install Azure Arc Agent - Allow remote connection to the trusted host
   Set-Item wsman:\localhost\Client\TrustedHosts -Value 192.168.0.4 -Force

   # Execute the script block on the remote machine
   Invoke-Command -ComputerName 192.168.0.4 -Credential $cred -ScriptBlock $block
   ```
 
1. Select the **SQLVM** resource and now you can see the dashboard of **SQLVM** SQL Server -Azure Arc from Azure Portal.

   ![](.././media/hybrid37.png "H1E3T2S8")

## Task 3: Run on-demand SQL Assessment.

With the SQL Server onboarded, this task walks you through setting up monitoring and assessment features. You’ll integrate the SQL Server with Log Analytics by installing the Log Analytics Agent extension. You will then configure SQL Server permissions and license type, and initiate a best practices assessment from the Azure Portal. This assessment provides a detailed evaluation of the SQL Server instance's configuration, helping identify security gaps, performance issues, and optimization recommendations.

1. Search for **Machines - Azure Arc (1)** from search box and click on **Machines - Azure Arc (2)**.
 
   ![](.././media/hyd20.png "server-azure-arc-search") 
   
1. Select **sqlvm** from the list of Azure Arc servers.

   ![](.././media/hyd21.png "select-sql-vm")
    
1. From the left navigation pane, expand **Settings (1)**, select **Extensions (2)** and click on the **+ Add (3)**.
 
   ![](.././media/new/r7.png)

1. Search for **Azure Monitor Agent (1)** extension press **Enter**, then select the **Azure Monitor Agent for Windows (Recommended) (2)** and click on the **Next (3)** button to continue.
 
   ![](.././media/new/r8.png)
   
1. On **Create** tab, click on **Review + create**.

   ![](.././media/new/r9.png)

1. Then click on **Create** to add an extension. 

   ![](.././media/new/r10.png)

   > **Note:** The deployment will take around 5 to 10 minutes to complete. You have to wait for this deployment to be successful to proceed to the next step.
   
1. From the desktop, open **Hyper-V Manager** and **double click** on **sqlvm** to connect to the Hyper-V sqlvm.

   ![](.././media/opensqlvm.png "opensqlvm")

1. On Connect to sqlvm box, scroll the bar towards **Small (1)** to open the VM in the smallest window and then click on the **Connect (2)** button.

   ![](.././media/new/t4.png)

1. Type password **demo@pass123** and press **Enter** button to login. Then, you can resize the SQLVM window at your convenience.
   
   ![](.././media/entervmpassword.png "entervmpassword")

1. Click on Start Menu and search for **Management**, then select **Microsoft SQL Server Management Studio 18**.
   
   ![](.././media/H1E3T3S13.png "H1E3T3S13")
  
1. On **Connect to server** pop-up, select **SQLVM (1)** as Server name from drop-down and click on **Connect (2)**.

   ![](.././media/arc23.png "H1E3T3S14")
   
1. In the left pane, expand **Security (1)** then **Logins (2)**. In Logins, right-click on **NT AUTHORITY\SYSTEM (3)** and click on **Properties (4)**.

   ![](.././media/arc24.png "H1E3T3S15")
  
1. In Login Properties pane, click on **Server Roles (1)** then enable the **sysadmin (2)** role and click on **Ok (3)**.

   ![](.././media/arc25.png "H1E3T3S16")
 
1. In **azure-arc** resouce group, search for **sqlvm (1)** and select **sqlvm - SQL Server Instance (2)**.

   ![](.././media/new/aa7.png)

1. From the left navigation pane, expand **Settings (1)**, select **Best practices assessment (2)**, select the log Analytics Workspace as **LogAnalyticsWS-<inject key="DeploymentID" enableCopy="false" /> (3)** from the drop-down and click on **Enable assessment (4)**.

   ![](.././media/new/aa8.png)

2. Once the assessment is enabled, refresh the page every 3 minutes until **Assessment Scheduled** is displayed. When it appears, click **Run Assessment (1)**. You should then see **Assessment in Progress**.

   >**Note:** The assessment in progress will take some time. Continue with the next exercise, and come back later to check. Once the assessment is completed, `click on the completed assessment` to view the results, as shown in the next step.

   ![](.././media/new/qq1.png)

   ![](.././media/new/qq2.png)

1. Once the assessment is **Completed**, click on it to see the results.

    ![](.././media/new/qq3.png)

1. The **Assessment results** will look like below:

    ![](.././media/new/qq4.png)
      
   > **Note:** Now you can move to the next Exercise, you don't have to wait here for the Result to appear.   

   > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
 
   - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task.
   - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
   - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.
 
<validation step="e9c36253-2523-4964-9996-0b518f38ee52" />

## Summary 

In this exercise, you registered an Azure Arc-enabled SQL Server, enabling centralized management and monitoring of SQL resources. You also performed an on-demand SQL Assessment to evaluate the server's configuration, identify potential issues, and receive recommendations for optimization and best practices.

### You have successfully completed the exercise. Click on **Next >>** from the bottom right corner to proceed with the next exercise.

 ![](.././media/arcg6.png)
