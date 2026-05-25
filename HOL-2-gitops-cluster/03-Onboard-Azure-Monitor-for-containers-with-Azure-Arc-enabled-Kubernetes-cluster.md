# Exercise 7: Onboard Azure Monitor for containers with Azure Arc-enabled Kubernetes cluster [Read-only]
### Estimated Duration: 30 Minutes
In this exercise, you will see how to configure Azure Monitor for containers and view insights for Kubernetes - Azure Arc resource.

## Objective

In this exercise, you will be performing the following task:

- Task 1: Configuring Azure Monitor

## Task 1: Configuring Azure Monitor

In this task, you will enable Azure Monitor for your Azure Arc-enabled Kubernetes cluster. You’ll link the cluster to a Log Analytics workspace, configure monitoring, and then explore cluster performance and insights using the Azure Portal. Full telemetry (nodes, pods, containers) becomes visible after some time.

1. Navigate to **azure-arc** resource group and select **microk8s-cluster** Kubernetes - Azure Arc resource from the resources listed.

   ![](.././media/new/e7.png)

1. On the **microk8s-cluster** pane, expand **Monitoring (1)** from the left navigation pane, select **Insights (2)** and click on **Configure monitoring (3)**.

   ![](.././media/new/e8.png)

   > **Note:** If the **Configure Monitoring** option is not visible, follow the steps below before proceeding:
   - **Step 1:** In the Azure portal, use the **Global Search bar**, search for **Monitor(1)**, and select **Monitor(2)** as shown in the image.

      ![](.././media/new/e8-1.png)
      
   - **Step 2:** On the **Monitor** page, navigate to **Insights (1)** --> **Containers (2)**. Select **Unmonitored clusters (3)**, locate the **microk8s-cluster** and click **Enable(4)**.
   
      ![](.././media/new/e8-2.png)


1. Under **Capabilities**, click on **Customize capabilities**.

   ![](.././media/new/e9.png)

1. For the Log Analytics workspace select the **loganalyticsws-<inject key="DeploymentID" enableCopy="false" />(2)** from the dropdown, from Logs presets dropdown select **Standard (2)** and click on **Save (3)**.

   ![](.././media/new/e10.png)

1. From the bottom, click on **Review + enable**.

   ![](.././media/new/ss2.png)

1. Then click on **Enable**.

   ![](.././media/new/ss3.png)

1. You will be able to see the insights data after `30-60 minutes`. For now, you can continue with the next **HOL** and come back later to review the insights.

1. In the Insights pane, refresh the page and filter the **Time range = Last 6 Hours (1)**. Click on **Cluster (2)** and review the insights. Now that your cluster is being monitored, you can watch the monitoring telemetry for the cluster, nodes and pods.

   ![](.././media/hol2-ex3-4.png "azuremonitor")

1. In the same pane, filter the **Time range = Last 6 Hours (1)** and click on **Nodes (2)** and select **ubuntu-k8s**. Here you can observe that the ubuntu-k8s server azure-arc node is listed below, which defines the integration of Azure Arc connected cluster with Azure Monitor for Containers.

   ![](.././media/hol2-ex3-5.png "azuremonitor")

   ![](.././media/hol2-ex3-6.png "azuremonitor")

7. In the same pane, filter the **Time range = Last 6 Hours (1)** and click on **Containers (2)**. You will be able to see the list of Containers that are linked to the pod and node which you have monitored in the previous steps.

   ![](.././media/hol2-ex3-7.png "azuremonitor")

## Summary

In this exercise, you configured Azure Monitor for containers to track performance and gain insights for an Azure Arc-enabled Kubernetes resource, enabling enhanced monitoring and visibility.

### You have successfully completed the exercise. Click on **Next >>** from the bottom right corner to proceed with the next exercise.

 ![](.././media/arcg6.png)