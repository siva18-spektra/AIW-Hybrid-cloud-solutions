# Hands-on Lab 02

# Exercise 5: Enable GitOps Configuration on connected K8s Cluster

### Estimated Duration: 60 Minutes
In addition to managing and monitoring their Kubernetes clusters, Contoso’s central development teams are building applications for internal inventory management at their distribution sites. They need these applications to be containerized and run on Kubernetes clusters. The locations are spread across the country, and Contoso is faced with the challenge of how to uniformly deploy, configure and manage their containerized applications across all these locations. By leveraging GitOps on Azure Arc-enabled Kubernetes, Contoso can centrally declare its Kubernetes configurations and applications in a Git repository and deploy them to all clusters simultaneously. Developers are more empowered because they can commit changes directly in the Git repo, and these updates are also automatically rolled out to all the clusters.

GitOps, as it relates to Kubernetes, is the practice of declaring the desired state of Kubernetes configuration (deployments, namespaces, etc.) in a Git repository, followed by a polling and pull-based deployment of these configurations to the cluster using an operator. In this exercise, you will deploy a sample Kubernetes app using the az k8sconfiguration command and gitops and also update the configuration in the repository which you have linked to the connected cluster and verify if the cluster is getting updated based on the changes made. You will be using the Kubernetes cluster with which you connected in the earlier exercise.

## Objectives

In this exercise, you will be performing the following tasks:

- Task 1: Fork the GitHub Arc K8s demo repository
- Task 2: Deploy App using az k8s configuration
- Task 3: Validate the Kubernetes configuration - **Read Only**

## Task 1: Fork the GitHub Arc K8s demo repository

In this task, you will create a personal copy (fork) of the public arc-k8s-demo GitHub repository. This will serve as the source for your GitOps deployment, allowing changes in your repo to reflect automatically in the connected Kubernetes cluster

1. Launch the following GitHub repository URL ```https://github.com/CloudLabsAI-Azure/arc-k8s-demo```. In the upper right corner, you will see **Sign in (1)** and **Sign up (2)** options. If you already have a github account, then click on **Sign in**, otherwise **Sign up**.

   ![](.././media/new/9.png)

   > **Note:** Please use your personal email address for GitHub.
   
1. If you click on **Sign in**, You will be prompted to provide your **Github Username/email address (1)** and Password (2) then click on **Sign in (3)**
   
   ![](.././media/hybrid47.png)
   
1. Then you will receive an **device verification code** to your email, enter that code **(1)** and then click on **Verify (2)**.

   ![](.././media/arc28.png)
   
1. Now, from the upper right corner, click on the **Fork** to fork the repository to your GitHub account.

   ![](.././media/new/a1.png)
   
1. On **Create a new fork**, uncheck the **Copy the** `master` **branch only (1)** and click **Create fork (2).**
   
   ![](.././media/new/a2.png)   

## Task 2: Deploy App using az k8s configuration

Here, you will log into the ubuntu-k8s VM and configure it using Azure CLI. You will connect the cluster to Azure Arc, install the Flux GitOps operator, and deploy a sample app by linking your forked GitHub repo. This sets up the GitOps automation pipeline for your cluster.

1. Using the Azure CLI extension for **k8sconfiguration**, link connected cluster to personal git repository. Provide this configuration a name **cluster-config**, instruct the agent to deploy the operator in the **cluster-config** namespace, and give the operator **cluster-admin** permissions. 

1. From the start menu of the **ARCHOST** VM, search for **putty (1)** and select **PuTTY (2)**.

    ![](.././media/new/a3.png)
     
1. In Putty Configuration tool, enter the **ubuntu-k8s** VM private IP - ```192.168.0.8``` **(1)**, make sure the Port value is ```22``` **(2)**. Once you have entered the private IP of the **ubuntu-k8s** VM, click on the **Open (3)** to launch the terminal.

    ![](.././media/arc3.png "Enter ubuntu-k8s VM private IP")
    
1. Enter the **ubuntu-k8s** vm username - ```demouser``` in **login as** and then hit **Enter**. 

   ```
   demouser
   ```

1. Now, enter the password - ```demo@pass123``` and press **Enter**. Remember password will be hidden and will not be visible in the terminal.

   ```
   demo@pass123
   ```

    ![](.././media/new/a4.png)
    
    > Note: To paste any value in the Putty terminal, just copy the value from anywhere and then right-click on the terminal to paste the copied value.

1. Log in with Sudo. Run the following command and provide the Password `demo@pass123`.

   ```
   sudo su
   ```
   
   ```
   demo@pass123
   ```
    
1. Next, you have to navigate back to the Desktop of the provided virtual Machine ARCHOST VM 💻, and then click on the `installArcAgentLinux.txt` file to open it.

   ![](.././media/new/a5.png)

1. Then, select the first 7 lines and, then right click and copy. 

1. Then, go back to the **putty session** and paste it into the ubuntu-k8s VM by doing a right click and it will start executing. 

   ![](.././media/variableazlogin.gif "Install Arc Agent")

1. Once it is executed, you have declared the values of AppID, AppSecret, TenantID, SubscriptionID, ResourceGroup, and location, and then logged into Azure using the 7th line. You can also find the values of these variables in the **Environment Details** tab. These variables are required for the next steps.

    ![](.././media/variableazlogin.png "azlogin")

1. Run the below commands one after the other.

   ```
   microk8s start
   microk8s status --wait-ready
   ```

    ![](.././media/gg-6-16.png "azlogin")   

     > **Note:** Wait until the first command runs successfully. This may take around **10–15 minutes**. Then, run the second command, which can take approximately **15–20 minutes** to complete.

     > **Note:** If `microk8s status --wait-ready` takes more than **20–30 minutes** to execute, press **Ctrl+Z** to terminate it and proceed further.

     > **Note:** If the resource `microk8s-cluster` shows **"Not Connected"** in the Azure Portal even after onboarding, you may need to manually refresh the cluster configuration and reconnect it to Azure Arc. Follow the steps below to resolve this:.
     >
     > ```bash
     > microk8s status
     > microk8s refresh-certs
     > cd $HOME
     > cd .kube
     > microk8s config > config
     > cd ..
     > az login -u $AppID --service-principal --tenant $TenantID -p $AppSecret
     > az connectedk8s connect --name microk8s-cluster --resource-group $ResourceGroup -l $location
     > ```
     >    ![](.././media/new-connectivity.png "connectivity")

     > After running these commands, make sure connectivity status is **Connected**, then wait a few minutes and then refresh the Azure Portal to verify that the connection status has changed to **Connected**. You can then continue with the next steps in the lab.

1. Run the below command to install `microsoft.flux` extension.

   ```
   az config set extension.dynamic_install=yes
   az config set extension.dynamic_install_allow_preview=true
   ```

   ```
   az k8s-extension create --extension-type microsoft.flux --configuration-settings multiTenancy.enforce=false -c microk8s-cluster -g $ResourceGroup -n flux -t connectedClusters
   ```
    >**Note:** Enter `Y` in `The command requires extension k8s-extension, Do you want to install`.

    >**Note:** If the command takes longer than **15 minutes** to run, terminate the process by pressing **Ctrl + Z**. The extension installation will continue in the Azure portal for adding the **Flux** extension to the **microk8s-cluster (Kubernetes - Azure Arc)**.

    >**Note:** If the execution fails, update your MicroK8s cluster by running the following command and retry Step 14:

    > ```
    > sudo snap refresh microk8s --channel=1.30/stable
    > ``` 

1. Once the previous command is executed successfully, the provisioning state in the output will show as **Succeeded**.

    ![](.././media/new/as5.png)

1. Copy the below command to any text editor. You have to replace **\<githubusername>** in the below command with the `username of the GitHub account` to which you had forked the repository.

   ```
   az k8s-configuration flux create   -g $ResourceGroup   -c microk8s-cluster   -n cluster-config   -t connectedClusters   --scope cluster   --namespace cluster-config   -u https://github.com/<githubusername>/arc-k8s-demo  --branch master --kustomization name=cluster-config-kustomization
   ```

    >**Note:** Enter `Y` to `The command requires extension k8s-configuration, Do you want to install`.

1. Once the previous command is executed successfully, the compliance state in the output will show as **Pending**:
   
    ![](.././media/cs.png) 
   
     > **Note:** Wait for 5 minutes before performing the next step

     > ``Info`` - Once you execute the above command, the manifests in your forked repository provision a few namespaces, deploy workloads and provide some team-specific configuration. Using this repository with GitOps creates the following resources on your Kubernetes cluster:

     > *Namespaces*: cluster-config, team-a, team-b
     
     > *Deployment*: cluster-config/arc-k8s

     > *ConfigMap*: team-a/endpoints
     
     > The config agent polls Azure for new or updated configurations.
  
## Task 3: Validate the Kubernetes configuration - Read Only

Now you will verify that the Kubernetes resources (like namespaces, deployments, and config maps) defined in your GitHub repository are deployed on the cluster. This confirms that GitOps is working and syncing properly.

   > ```Info```: Flux is the operator that makes GitOps happen in your cluster. It ensures that the cluster config matches the one in git and automates your deployments.

1. To verify that the namespaces, deployments, and resources are created, **run the following command** in the SSH Session opened to the ubuntu-k8s VM from Putty:

   ```
   kubectl get ns --show-labels
   ```
 
   The output shows that `team-a, team-b, and cluster-config` namespaces have been created as shown:
  
   ![](.././media/arc35.png)
   
1. You can explore the other resources deployed as part of the configuration repository by running the following commands:

   ```
   kubectl -n team-a get cm -o yaml
   ```

## Summary

In this exercise, you deployed a sample Kubernetes app using the az k8sconfiguration command and GitOps. You then updated the linked repository configuration and verified that the connected Kubernetes cluster applied the updates automatically based on the changes made.

### You have successfully completed the exercise. Click on **Next >>** from the bottom right corner to proceed with the next exercise.

 ![](.././media/arcg6.png)
