# Hands-on Lab 03
# Exercise 8: Deploying Azure Arc Data Controller with direct connectivity mode and Azure Arc-enabled SQL Managed Instance Business Critical
### Estimated Duration: 90 Minutes  
In this exercise, you will be connecting an existing Kubernetes cluster to Azure using Azure Arc-enabled Kubernetes. You will be deploying an Azure data controller in direct connectivity mode to a custom location using Azure portal and Azure CLI, and later you will be creating the Azure Arc-enabled SQL Managed Instance Business Critical on top of the Azure Arc Data Controller. In the short term, you will be preparing an infrastructure for the next exercise to restore the Databases into the Azure SQL Managed Instance. 

## Objectives

In this exercise, you will be performing the following tasks:

- Task 1: Log in to Azure and install Azure CLI extensions.
- Task 2: Onboard an existing Kubernetes cluster to Azure using Azure Arc-enabled Kubernetes.
- Task 3: Create a custom location on the Azure Arc-enabled Kubernetes cluster
- Task 4: Deploy Azure Arc Data Controller in directly connected mode using Azure Portal.
- Task 5: Monitor the creation of Azure Arc data controller on the cluster.
- Task 6: Deploy Azure Arc-enabled SQL Managed Instance using Azure Portal.
- Task 7: Connecting Azure Arc Data Controller using Azure Data Studio.[Read-only]
- Task 8: Connect to Azure Arc-enabled SQL Managed Instance using Azure Data Studio.[Read-only]


## Task 1: Log in to Azure and install Azure CLI extensions.

In this task, you will connect a running Azure Kubernetes Service (AKS) cluster to Azure using Azure Arc. You’ll log in to Azure, install and update required CLI extensions, register necessary resource providers, and run the onboarding command to register the Kubernetes cluster with Azure Arc.

1. Open **Windows PowerShell** by double-clicking on the **Windows PowerShell** icon from the desktop of your ARCHOST VM

    ![](././media/new/1.png)
    
1. Run the below command to log in to Azure.

    ```
    az login
    ```
    
    >**Note:** If the `az login` gives a AssertionError, run the following command.Later again run the `az login` command and continue with the next step

    ```
    Remove-Item -Recurse -Force "C:\Users\arcadmin\.azure\cliextensions\arcdata"
    az extension add --name arcdata
    ```

1. After running the above command, a browser tab will open to log in to the Azure portal.

1. On the **Sign into Microsoft Azure** tab, you will see the login screen. Enter the following **Email/Username** and then click on **Next**.

   * Email/Username: **<inject key="AzureAdUserEmail"></inject>**

1. Now enter the following **Password** and click on **Sign in**.

   * Password: **<inject key="AzureAdUserPassword"></inject>**

1. After adding the credentials, you will see that you have logged into Microsoft Azure.

    ![](media/new/4.png)

1. Now switch back to Windows PowerShell and you will be able to see that you have logged in to Azure.

1. Navigate to **arcadmin** using the below command.
   
   ```
    cd C:\Users\arcadmin 
   ```      

1. Run the below commands to install the required Azure CLI extensions.
   
   ```
   az extension add --name k8s-extension
   az extension add --name connectedk8s
   az extension add --name k8s-configuration
   az extension add --name customlocation   
   ```
   
   >**Note:** If you get any warnings, please ignore. They are not errors.
   
1. Now run the below command to get the latest version of extensions.
  
   ```
   az extension update --name k8s-extension
   az extension update --name connectedk8s
   az extension update --name k8s-configuration
   az extension update --name customlocation
   az extension update --name arcdata 
   ```

    >**Note:** If you get any warnings, please ignore. They are not errors.   
   
1. You can validate that you have all the required extensions with the latest versions by running the below command:
   
   ```
   az version
   ```
   
   ![](media/new/2.png)
   
1. After confirming that the required tools are installed, the next step is to register your subscription with Arc for Kubernetes.

1. Run the below commands to register the required resource providers if not already registered.

   ```
   az provider register --namespace Microsoft.Kubernetes
   az provider register --namespace Microsoft.KubernetesConfiguration
   az provider register --namespace Microsoft.ExtendedLocation
   ```
   
   ![](media/new/3.png)
   
## Task 2: Onboard an existing Kubernetes cluster to Azure using Azure Arc-enabled Kubernetes

In this task, you will be connecting an existing Kubernetes cluster to Azure using Azure Arc-enabled Kubernetes and will be enabling custom features by adding an Azure Arc data services extension and a custom location on the Azure Arc-enabled Kubernetes cluster.

1. Run the below command to import the **Kubernetes cluster credentials** in the environment.

   ```
   Import-AzAksCredential -ResourceGroupName $env:resourceGroup -Name Arc-Data-Demo-DirectMode -Force
   ```

1. Run the below command to connect the **existing Kubernetes to your Azure subscription** using Azure Arc-enabled Kubernetes. Once you have run the command, it will take a few minutes to onboard the cluster to Azure Arc.

   ```
   az connectedk8s connect --name Arc-Data-Demo-DirectMode --resource-group azure-arc
   ```

   > **Info:** In the above step, you have connected your existing Azure Kubernetes cluster to Azure Arc. To get the available AKS cluster deployed on Azure, you can use the command ```az aks list```.
  
   > **Note:** We have already defined your Cluster name and Azure resource group name in the above commands. If you are trying this in your subscription, please make sure that you have entered the correct details. `This can take up to 5 minutes to complete`. 

1. Once the previous command is executed successfully, the provisioning state in output will show as **Succeeded**.
   
   ![](media/new/5.png)

1. Verify whether the Azure Arc-enabled Kubernetes cluster is onboarded and connected to the resource group in the Azure subscription by running the following command:

   ```
   az connectedk8s list -g azure-arc -o table
   ```  
  
    ![](media/new/6.png)

     >**Note:** If you encounter the following error, run the commands below to complete the Azure CLI installation process, and then try executing the previous command again.
    
    ```
    az extension update --name connectedk8s
    ```
    ![](.././media/3.png)

   > **Note:** if you get a warning as below, kinldy run `choco install azure-cli -y` and continue udpating the az cli version and retry the above step by opening a new powershell terminal. This installation might take 5-7 mins.    
   >    ![](.././media/az-upgrade.png)
   
1. Azure Arc-enabled Kubernetes deploys a few operators into the azure-arc namespace. You can view these deployments and pods by running the command in the command prompt:  

    > **Note:** A Kubernetes operator is an application-specific controller that extends the functionality of the Kubernetes API to create, configure, and manage instances of complex applications on behalf of a Kubernetes user.

   ```
   kubectl -n azure-arc get deployments,pods
   ```
   
   The output should be similar to as shown below:    
  
   ![](media/new/8.png)

1. Navigate to **azure-arc** resource group. Click on **Arc-Data-Demo-DirectMode** of the resource type **Kubernetes - Azure Arc**.

   ![](media/new/7.png)
     
1. You can see the status as **Connected**.

   ![](media/hybrid60.png)

## Task 3: Create a custom location on the Azure Arc-enabled Kubernetes cluster

In this task, you will enable the necessary Arc features on the Kubernetes cluster, deploy the Arc Data Services extension, and create a custom location in Azure. This custom location allows Azure services to treat your cluster as a target region for deploying data services like SQL Managed Instance. Custom Locations provides administrators a way to deploy Azure Arc data services and other Azure Arc-enabled services to their locations, similar to Azure locations.

1. Now run the below command to enable features to create the **custom location:**

     ```
     az connectedk8s enable-features -n Arc-Data-Demo-DirectMode -g azure-arc --features cluster-connect custom-locations
     ```
     
    The output should be similar to as shown below:**"Successfully enabled features: ['cluster-connect', 'custom-locations'] for the Connected Cluster Arc-Data-Demo-DirectMode"**
   
    ![](media/new/9.png)
        
   > **Note:** The Custom Locations feature is dependent on the Cluster Connect feature. So, both features have to be enabled for custom locations to work. Also, az connectedk8s enable features need to be run on a machine where the kubeconfig file is pointing to the cluster on which the features are to be enabled.
    
1. Run the below command to deploy the extension of Azure Arc-enabled Data Services on the Azure Arc Kubernetes cluster.
  
     ```
    az k8s-extension create --name azdata --extension-type microsoft.arcdataservices --cluster-type connectedClusters -c Arc-Data-Demo-DirectMode -g azure-arc --scope cluster --release-namespace azure-arc --config Microsoft.CustomLocation.ServiceAccount=sa-bootstrapper --auto-upgrade false
     ```

   > **Note:** The above command will take up to 5 minutes to complete the creation of azdata extension, please wait for it to complete before proceeding to the next task.

1. After running the above command you will notice that the **Provisioning State** is **Succeeded**. If it is pending, it is because the extension may take a few minutes to complete the installation.
   
    ![](media/new/12.png)

1. To verify the extension installation, navigate back to azure-arc resource group, in the search bar, search for **Kubernetes - Azure Arc (1)** and select **Arc-Data-Demo-DirectMode (2)**.
   
    ![](media/new/10.png)
   
1. From the left navigation, expand **Settings (1)**, select **Extension (2)** and check if the Install status is **Succeeded (3)** or not. If it is not, please refresh after some time and then check.
   
    ![](media/new/11.png)

1. Now run the below command to get the Azure Resource Manager identifier of the Azure Arc-enabled Kubernetes cluster, you will be using the cluster-ID in the later steps while creating the custom location.

     ```  
     $clusterID = az connectedk8s show -n Arc-Data-Demo-DirectMode -g azure-arc  --query id -o tsv
     $clusterID
     ```
     
   > **Note:** The clusterID is stored in the $clusterID parameter and you will be using this parameter only in the later steps.
       
    ![zx](media/clusterid.png)
    
1. Now run the below command to get the Azure Resource Manager identifier of the cluster extension deployed on top of the Azure Arc-enabled Kubernetes cluster, referenced in the later steps as extensionId:

     ```
     $extensionID = az k8s-extension show --name azdata --cluster-type connectedClusters -c Arc-Data-Demo-DirectMode -g azure-arc --query id -o tsv
     $extensionID
     ``` 

   > **Note:** The extension resource ID is stored in the $extensionID parameter and you will be using this parameter only in the later steps.  
   
   ![sad](media/extensionid.png)
    
1. Now run the below command to create a custom location by referencing the Azure Arc-enabled Kubernetes cluster ID and the extension ID.

    ```  
    az customlocation create -n azurearc-nyc-location -g azure-arc --namespace azure-arc --host-resource-id $clusterID --cluster-extension-ids $extensionID
    ```
    
   > **Note:** This can take up to 2 minutes to complete the creation of a custom location. The output should be as shown below:    
    
    ![dfs](media/arc41.png)
     
1. To verify the custom location deployment, switch back to the browser and log in to [Azure Portal](https://portal.azure.com) if not already done.

1. Search for **custom location (1)** in the search bar and select **custom locations (2)**.
   
    ![sdf](./media/arc42.png)
      
1. After selecting the custom locations from the search bar, select your **azurearc-nyc-location**.

    ![](media/new/13.png)
     
1. Explore the overview section. You can see the namespace and Kubernetes cluster details on the overview page.
  
    ![](media/new/14.png)

1. Now search for the **Log Analytics workspace (1)** in the Azure portal and select **Log Analytics workspace (2)**.
     
    ![](./media/arc44.png)

1. Navigate to **LoganalyticsWS-Direct** workspace.
  
    ![](./media/hybrid65.png)

1. From the **Overview** page, copy the Worspace id in a notepad file for later use while creating the Azure arc data controller.

    ![](media/new/15.png)

1. Open **CloudShell** from the Azure portal.

   ![](media/new/q1.png)

1. Click on **Powershell (1)**, then select **No storage account required (2)** and choose the **default subscription (3)**.Click on **Apply (4)**

   ![](media/new/q2a.png)

   ![](media/new/q3a.png)

1. Now, in the powershell window, paste the following command
   
      ```
       Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName "azure-arc" -Name "LoganalyticsWS-Direct"
    
      ```  
1.  Copy the value of the primary key in a notepad file and proceed with the next task. 

    ![](./media/pmkey.png)
    
## Task 4: Deploy Azure Arc Data Controller in directly connected mode using Azure Portal

In this task, you will deploy an Azure Arc Data Controller to the custom location using the Azure Portal. You will configure parameters like service type, credentials, and workspace integration, enabling centralized monitoring and management for data services hosted on your cluster.

1. From the Azure Portal, search for **Azure arc data controllers (1)** from the search box and then select **Azure arc data controllers (2)**.  

    ![](media/new/q4.png)

1. After selecting the Azure Arc data controller, click on the **+ Create** button to deploy ```Azure arc data controller```.

    ![](./media/dc-2.png)
     
1. Now, on ```Create Azure Arc data controller``` blade, select **Azure Arc-enabled Kubernetes (Direct connectivity mode) (1)**. and click on **Next: Data Controller details (2)**.

    ![](./media/hybrid66.png)
   
1. On the **Data controller details** blade, enter the following details:

   * Select the available subscription from the dropdown **(1)**.

   * Resource Group: Select **azure-arc (2)** from dropdown.

   * Data Controller Name: **arcdc-direct (3)**

   * Custom location: Select the available custom location from dropdown **(4)**.

        ![](./media/arc47.png)
      
1. Now scroll down and enter the below details in the remaining sections.
   
   Under Kubernetes configuration, enter the details below:
   
   * Kubernetes configuration template: Select **azure-arc-aks-default-storage (1)** from dropdown.

   * Data Storage class: Leave default

   * Log Storage class: Leave the default

   * Service type: **Load balancer (2)**
   
   Under the Metrics and Logs Dashboard Credentials enter the below details.

   * Username: **arcuser (3)**

   * Password: **Password.1!! (4)**

   * Confirm password: **Password.1!! (5)**

   After entering all the required details, click on **Next: Additional settings** (6)

    ![](./media/dc-5.png)

1. In the Additional settings blade, select the **LoganalyticsWS-Direct** **(1)** from the dropdown for the Log Analytics workspace. You will see that the Log Analytics workspace ID occurs by default. Enter the **Log Analytics primary key** **(2)** which you have copied to Notepad earlier in task 2 and click on the **Next: Tags (3)** button.
   
    ![](./media/dc-6.png)
    
1. Leave default on **Tags** blade and click on **Next: Review + Create** button. to start the Azure Arc data controller deployment.
  
    ![](./media/dc-7.png)

1. On Review + Create Blade, you can check all the given details and click on the **Create** button to start the Azure Arc data controller deployment.

    > **Note:** The deployment of the Azure Arc data controller can take up to 10 minutes to complete.
  
    ![](./media/new/q5.png)
   
1. Once the deployment is completed, click on the **Go to resource group** button.
 
    ![](./media/complete-dc-direct.png)
   
1. From the **azure-arc** resource group, select **arcdc-direct** Azure Arc Data Controller from the resources.
 
    ![](./media/rg-dc-direct.png)  
   
## Task 5: Monitor the creation of Azure Arc data controller on the cluster.

In this task, you will monitor the deployment of the Azure Arc Data Controller on your Kubernetes cluster. You’ll use kubectl to check deployment status, and once the controller shows as “Ready,” you can proceed to deploy Arc-enabled data services.
   
1. When the Azure portal deployment status shows the deployment was successful, you can check the status of the Arc data controller deployment on the cluster by running the below command on the PowerShell window:

   ```
   kubectl get datacontrollers -n azure-arc
   ```  
    > **Note:** The deployment of the Azure Arc Data Controller can take 10–15 minutes to complete. If it takes significantly longer, please delete the `arcdc-direct` Azure Arc Data Controller and redeploy.
  
    ![](./media/status-dc-direct.png)
   
1. Once the data controller state is changed to ready, proceed to the next steps. Please note that the data controller deployment can take `5-to-10 minutes` to change it to ready.

1. On the Azure Ac data controller resource overview blade, explore the given information about the Namespace and Connection mode.
  
    ![](./media/hybrid67.png)

## Task 6: Deploy Azure Arc-enabled SQL Managed Instance using Azure Portal.

Let's create an **Azure Arc-enabled SQL Managed Instance** using Azure Portal on a directly connected Azure Arc data controller in a custom location.

1. Open your browser and log in to the Azure portal if not already logged in.

1. Now search for **SQL Managed Instance - Azure Arc (1)** and select **SQL Managed Instance - Azure Arc (2)**.
   
    ![](./media/arc48.png)
   
1. Click on the **+ Create** button to create the SQL Managed instance - Azure Arc.

    ![](./media/sqlman-2.png)

1. Now on the **Basics** tab, enter the below details:

   - **Under Project details**
    
     - **Subscription:** Leave ``default (1)``
    
     - **Resource Group:** Select **azure-arc (2)** from drop-down     
   
   - **Under Managed Instance details**
   
     - **Instance name:** Enter **arcsql-direct (3)**
  
     - **Custom location:** Select available custom location from drop-down **(4)**.
   
     - **Service type:** Select **Load balancer (5)** from drop-down.
    
     - **Compute + Storage:** Click on **Configure compute + storage (6)**
      
        ![](./media/arc49.png)
      
1. Now on **Compute + Storage** blade enter the following details:
    
     - Service Tier: **Business Critical (1)**
       
     - For Development use only: **Check the box (2)**
       
     - License Type: **License Included (3)**
       
        ![](./media/arcbasiclicense.png)
       
     - High availability: Select **2 replicas (4)**
       
     - Readable secondary replicas: Enter **1 (5)**

     - Instance Compute
       
       - Memory Request (in Gi): Enter ```4``` **(6)**
   
       - CPU vCores Request: Enter ```2``` **(7)**
  
       - Memory Limit (in Gi): Enter ```4``` **(8)**
         
       - CPU vCores Limit: Enter ```2``` **(9)**
         
          ![](./media/new/ss1.png)
         
     - Instance Storage

       - Data storage class: leave default

       - Data volume size (in Gi): ```2``` **(10)**
      
       - Data-logs storage class: leave ```default```

       - Data-logs volume size (in Gi): ```1``` **(11)**

       - Logs storage class: Leave ```default```

       - Logs volume size (in Gi): Enter ```1``` **(12)**

       - Backup Storage class: leave ```default```

       - Backups volume size (in Gi): ```1``` **(13)**

      >**Note:** In the above section the (Gi) is referring to the storage in Gigabytes.  
      
    After adding all the above details, click on the **Apply (14)** button.  
        
      ![](./media/new/ss2.png)
   
1. Under the Administrator account, enter the below details:

     - **Managed Instance admin login:**  Enter `arcsqluser` **(1)**
   
     - **Password:** Enter `Password.1!!` **(2)**
     
     - **Confirm Password:** Enter `Password.1!!` **(3)**

   After adding all the required details, click on the **Review + Create (4)** button to review all details.

   ![sds](./media/sqlman-6.png)
    
1. Now, click on the **Create** button to start the deployment.  
   
    ![](./media/review-sqlmi-direct.png)

1. After some time, you will see that the deployment of **SQL Managed Instance - Azure Arc** is completed. Now click on the **Go to resource** button to navigate to the resource.
  
    ![](./media/complete-sqlmi-direct.png)

1. Now we have successfully deployed the Azure Arc-enabled SQL Managed Instance on top of the Directly connected mode Azure Arc data controller, you can explore more on metrics and logs on the same page from the left side menu. Please note that Azure Arc-enabled SQL Managed Instance deployment can take 5 to 10 minutes to change it to ready.
  
    ![](./media/hybrid69.png)

## Task 7: Connecting Azure Arc Data Controller using Azure Data Studio [Read-only]

Now, let us connect to the data controller using Azure Data Studio.

1. Let us download the latest Windows 64-bit Python version by running the below commands in Windows PowerShell (run as administrator).

   ```
   choco install wget
   wget https://www.python.org/ftp/python/3.13.1/python-3.13.1-amd64.exe -OutFile "C:\Users\arcadmin\Downloads\python-3.13.1-amd64.exe"
   ```
   >**Note:** You can also download the file manually by navigating to the official website - https://www.python.org/downloads/ 

1. Run the below command to install pip.

   ```
   Start-Process -FilePath "C:\Users\arcadmin\Downloads\python-3.13.1-amd64.exe" -ArgumentList "/quiet InstallAllUsers=1 PrependPath=1 Include_pip=1" -Wait
   ```

1. Open **Azure Data studio** **(1)** from the desktop shortcut, select **Connections** **(2)** and click on **Connect to Existing Azure Arc Controller** **(3)**.
  
    ![](./images/15-05-2024(2).png)
   
2. On the Connect to Existing Controller page, provide the following details and click on **Connect (4)**.

   - **Namespace (1):**
     ```BASH
     azure-arc
     ```
     
   - **Cluster Context (2):**
     ```BASH    
     Arc-Data-Demo-DirectMode
     ```
     
   - **Name** : Enter **arcdc-direct (3)**
     ```BASH
     arcdc-direct
     ```  
        ![asdasd](./media/arc52.png)

        >**Note:** If you see any error message in the **Cluster Context**, open **PowerShell** and run the commands below one by one. After that, close and reopen **Azure Data Studio** from the desktop, and then repeat Step 4.

        ```BASH
        Remove-Item C:\Users\arcadmin\.kube\config
        ```

        ```BASH
        Import-AzAksCredential -ResourceGroupName $env:resourceGroup -Name Arc-Data-Demo-DirectMode -Force
        ```

3. Once the connection is successful, you can see the Azure Arc data controller listed under Azure Arc Controllers on the bottom left of the Azure Data Studio.
   
    ![](./media/ads-direct-list.png)

5. Right-click on the **arcdc-direct (1)** Azure Arc Controller and select **Manage (2)**.
  
    ![](./media/new/q6.png)

6. Once you are in the Azure Arc Data Controller dashboard, you can see the following details about the data controller

   - Name of the Arc Data Controller

   - Region where it is deployed

   - Connection mode

   - Resource Group

   - Subscription ID of the Azure Subscription

   - Controller Endpoint

   - Namespace
   
   You will also see that we have deployed using the Direct connection mode of the Azure Arc Data controller.
  
    ![](./media/ads-direct-overview.png "Azure Data Studio")

## Task 8: Connect to Azure Arc-enabled SQL Managed Instance using Azure Data Studio. [Read-only]

In this task, let us learn how to connect to Azure Arc-enabled SQL Managed instance using Azure Data Studio.

1. You can see the Azure Arc-enabled SQL Managed Instance named **arcsql-direct** listed under Azure Arc Data Controller named **arcdc-direct** at the bottom left of the Azure Data Studio. Right-click on the **arcsql-direct** and select **Manage**.
  
    ![](./media/ads-sqlmi-manage.png)

1. Once you are in the SQL-managed instance - Azure Arc dashboard, you can see the following details about the data controller:

   - Name of the Resource Group

   - Name of the Arc Data Controller

   - Subscription ID of the Azure Subscription

   - External Endpoint

   - Status of SQL Managed Instance

   - Region where it is deployed

   - Compute: Number of vCores

   Make sure you copy the **External Endpoint with port number** and save it in a notepad for use later in the task.   
  
    ![](./media/ads-sqlmi-overview.png)

    >**Note:** Please make sure the Managed instance is ready and is showing the `Managed Instance Admin`.
    
1. In the Connections tab of Azure Data Studio, within the servers, click on **New Connection**.
  
    ![](images/arc53.png "Confirm")

1. Enter the following on the connection details page:

   - **Connection type** : Select **Microsoft SQL Server (1)**
   
   - **Server:** Paste the External Endpoint value of SQL Managed Instance which you copied earlier **(2)**

     >**Note:** Make sure you have entered **IP Address** with **port number**.
   
   - **Authentication type** : Select **SQL Login** from the drop-down options **(3)**
   
   - **User name** : Enter arcsqluser **(4)**

     ```BASH
     arcsqluser
     ```
   
   - **Password** : Enter Password.1!! **(5)**
     ```BASH
     Password.1!!
     ```

   - **Trust server Certificate** : Mark it as **True (6)**
   - Leave the other values as default.
   - Then click on **Connect (7)**
  
    ![](./media/HOL23-Ex8.png)

    >**Note:** If you see a connection error message then click on **Enable trust server certificate.**   
   
1. Now you can see that you are successfully connected with your Azure Arc-enabled SQL MI Server. You can see it listed under **Servers**. You can explore the SQL Managed Instance - Azure Arc Dashboard to view the databases and run a query.
  
    ![](./media/ads-9.png)


> **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
 
- Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task.
- If not, carefully read the error message and retry the step, following the instructions in the lab guide.
- If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.
 
<validation step="9864336d-56ad-4653-9737-e6f3ab34de18" />

## Summary

In this exercise, we connected our cluster to the Azure Arc-enabled cluster and deployed a custom location and data controller with directly connected mode with the help of Azure portal and Azure CLI, and created an Azure Arc-enabled SQL Managed Instance server on the directly connected mode of Azure Arc data controller. Also, we have connected the Azure Arc Data Controller and Azure Arc-enabled SQL Managed Instance Business Critical using Azure Data Studio.

### You have successfully completed the exercise. Click on **Next** from the bottom right corner to proceed with the next exercise.

 ![](.././media/arcg6.png)
