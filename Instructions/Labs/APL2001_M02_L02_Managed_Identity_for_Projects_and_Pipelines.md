---
lab:
    title: 'Managed identity for projects and pipelines'
    module: 'Module 2: Manage identity for projects, pipelines, and agents'
---

# Managed identity for projects and pipelines

Managed identities offer a secure method for controlling access to Azure resources. Azure handles these identities automatically, allowing you to verify access to services compatible with Azure AD authentication. This means you won't need to embed credentials into your code, enhancing security. In Azure DevOps, managed identities can authenticate Azure resources within your self-hosted agents, simplifying access control without compromising security.

In this lab, you'll create a managed identity to use in your YAML pipelines using Azure DevOps with self-hosted agents and a managed identity.

These exercises take approximately **45** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

- Verify that you have a Microsoft account or an Azure AD account with the Contributor or the Owner role in the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Instructions

### Exercise 1: Import and run CI/CD Pipelines

In this exercise, you will import and run the CI pipeline, configure the service connection with your Azure Subscription and then import and run the CD pipeline.

#### Task 1: (If done, skip) Import and run the CI pipeline

Let's start by importing the CI pipeline named [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the eShopOnWeb project.
3. Go to **Pipelines > Pipelines**.
4. Click on **New Pipeline** button.
5. Select **Azure Repos Git (Yaml)**.
6. Select the **eShopOnWeb** repository.
7. Select **Existing Azure Pipelines YAML File**.
8. Select the **/.ado/eshoponweb-ci.yml** file then click on **Continue**.
9. Click the **Run** button to run the pipeline.
10. Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-ci** and click on **Save**.

#### Task 2: Manage the service connection

You can create a connection from Azure Pipelines to external and remote services for executing tasks in a job.

In this task, you will create a service principal by using the Azure CLI, which will allow Azure DevOps to:

- Deploy resources on your azure subscription
- Deploy the eShopOnWeb application

> [!NOTE]
> If you do already have a service principal, you can proceed directly to the next task.

You will need a service principal to deploy Azure resources from Azure Pipelines.

A service principal is automatically created by Azure Pipeline when you connect to an Azure subscription from inside a pipeline definition or when you create a new service connection from the project settings page (automatic option). You can also manually create the service principal from the portal or using Azure CLI and re-use it across projects.

1. Start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has the Owner role in the Azure subscription you will be using in this lab and has the role of the Global Administrator in the Azure AD tenant associated with this subscription.
2. In the Azure portal, click on the **Cloud Shell** icon, located directly to the right of the search textbox at the top of the page.
3. If prompted to select either **Bash** or **PowerShell**, select **Bash**.

   > [!NOTE]
   > If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**.

4. From the **Bash** prompt, in the **Cloud Shell** pane, run the following commands to retrieve the values of the Azure subscription ID attribute:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > [!NOTE]
    > Copy both values to a text file. You will need them later in this lab.

5. From the **Bash** prompt, in the **Cloud Shell** pane, run the following command to create a service principal:

    ```sh
    az ad sp create-for-rbac --name sp-eshoponweb-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > [!NOTE]
    > The command will generate a JSON output. Copy the output to text file. You will need it later in this lab.

6. Next, navigate to the Azure DevOps **eShopOnWeb** project. Click on **Project Settings > Service Connections (under Pipelines)** and **New Service Connection**.

7. On the **New service connection** blade, select **Azure Resource Manager** and **Next** (may need to scroll down).

8. Then choose **Service principal (manual)** and click on **Next**.

9. Fill in the empty fields using the information gathered during previous steps:
    - Subscription Id and Name.
    - Service Principal Id (or clientId), Key (or Password) and TenantId.
    - In **Service connection name** type **azure subs**. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.

10. Click on **Verify and Save**.

#### Task 3: (If done, skip) Import and run the CD pipeline

Now, import the CD pipeline named [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Go to **Pipelines > Pipelines**.
2. Click on **New pipeline** button.
3. Select **Azure Repos Git (Yaml)**.
4. Select the **eShopOnWeb** repository.
5. Select **Existing Azure Pipelines YAML File**.
6. Select the **/.ado/eshoponweb-cd-webapp-code.yml** file then click on **Continue**.
7. In the YAML pipeline definition, in the variables section, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
   - **az400eshop-NAME**, with a web app name to be deployed with a global unique name, for example, "eshoponweb-lab-YOURNAME".
   - **AZ400-EWebShop-NAME** with the name of your preference, for example, "rg-eshoponweb".

8. (Optional) You can use a self-hosted agent updating the pool name currently set to the Microsoft-hosted agent to the name of the agent pool you created, "eShopOnWebSelfPool".

    Instead of:

    ```YAML
        pool:
          vmImage: windows-latest
    
    ```

    Use:

    ```YAML
        pool: eShopOnWebSelfPool
    
    ```

    > [!NOTE]
    > To run the pipeline with the self-hosted agent, you will need to have the agent running and all the prerequisites installed, for example, Visual Studio to build the solution. If you do not have the prerequisites installed, you can use the Microsoft-hosted agent.

9. Click on **Save and Run**, choose to commit directly to the main branch, or create a new branch.
10. Click on **Save and Run** again.

    > [!NOTE]
    > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

11. Open the pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", click on **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.

    ![Screenshot of the permit access from the YAML pipeline.](media/pipeline-deploy-permit-resource.png)

12. The deployment may take a few minutes to complete, wait for the pipeline to execute. The CD definition consists of the following tasks:
      - **Resources**: it is prepared to automatically trigger based on CI pipeline completion. It also downloads the repository for the bicep file.
      - **AzureResourceManagerTemplateDeployment**: Deploys the Azure Web App using bicep template.
13. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-cd-webapp-code** and click on **Save**.

### Exercise 2: Create a managed identity for the service connection

In this exercise, you will create a managed identity and then create a new service connection to use it in the CI/CD pipelines.

#### Task 1: Create a managed identity

1. In your browser, open the [**Azure Portal**](https://portal.azure.com).
2. In the "Search resources, services and docs (G+/)" box, type "Managed Identities" and select it from the dropdown list.

    ![Screenshot of the Managed Identities option in the Azure Portal.](media/managed-identities.png)

3. Click on the "Create managed identity" button.
4. In the "Create Managed Identity" pane, fill in the required information:
   - **Subscription** with your Azure subscription.
   - **Resource Group** with existing or new resource group.
   - **Region** with the region close to your location or available for your resources.
   - **Name** with the Managed Identity name of your preference, for example, "eshoponweb-mi".

    ![Screenshot of the create Managed Identity pane.](media/create-managed-identity.png)

    > [!NOTE]
    > If you don't have a resource group, you can create one by clicking on the "Create new" link.

5. Click on the "Review + create" button, then click "Create".

#### Task 2: Assign permissions to the Managed Identity

Next, you need to assign the Managed Identity permissions to the resource group and app services.

1. In the Azure portal, navigate to the new Managed Identity you created earlier.
2. Click on the Azure role assignments tab.
3. Click on the "Add role assignment" button.
4. Select "Resource Group" as the scope.
5. Select the subscription and resource group you used in the previous steps.
6. Select the "Contributor" role.
7. Click on the "Save" button.

    ![Screenshot of the add role assignment pane.](media/add-role-assignment.png)

### Exercise 3: Create a new Azure Virtual Machine using self-hosted agent and the Managed Identity and update the CI pipeline

In this exercise, you will create a new Azure Virtual Machine using the self-hosted agent and the Managed Identity you created in the previous exercise. Then, you will update the CI pipeline to use the new Azure Virtual Machine.

#### Task 1: Create a new Azure Virtual Machine

1. In your browser, open the [**Azure Portal**](https://portal.azure.com).
2. In the "Search resources, services and docs (G+/)" box, type "Virtual Machines" and select it from the dropdown list.
3. Click on the "Create" button.
4. Click on "Azure virtual machine with preset configuration".

    ![Screenshot of the create virtual machine with preset configuration.](media/create-virtual-machine-preset.png)

5. Select the "Dev/Test" as the workload environment and the "General purpose" as the workload type.
6. Click on the "Continue to create a VM" button.
7. Fill in the required information:
   - **Subscription** with your Azure subscription.
   - **Resource Group** with existing or new resource group, for example, "eshoponweb-resource".
   - **Virtual machine name** with the name of your preference, for example, "eshoponweb-vm".
   - **Region** with the region close to your location or available for your resources, for example, "South Central US".
   - **Availability options** with "No infrastructure redundancy required".
   - **Security type** with the "Trusted launch virtual machines" option.
   - **Image** with the "Windows Server 2019 or 2022 Datacenter" image.
   - **Size** with the cheapest "Standard" size for testing purposes.
   - **Username** with the username of your preference.
   - **Password** with the password of your preference.
   - **Public inbound ports** with "Allow selected ports".
   - **Select inbound ports** with "RDP (3389)".
   - **Public IP address** with "Create new".

    > [!IMPORTANT]
    > Don't skip the step Exercise 5: Remove the Azure lab resources to avoid unexpected charges.

8. Click on the "Management" tab and check the "Enable system assigned managed identity" option. This will allow the VM to use the Managed Identity you created.
9. Click on the "Review + create" button, then click "Create".
10. Open the virtual machine settings, click on the "Identity" tab and click on the "Azure role assignments" button.
11. Click on the "Add role assignment" button.
12. Select the subscription scope, subscription and the "Contributor" role.
13. Click on the "Save" button.

#### Task 2: Open the new Azure Virtual Machine and install the self-hosted agent

1. Open the new Azure Virtual Machine you created earlier using the RDP connection. You can find the connection information in the "Overview" checking the "Connect" button.
2. From the Azure VM, follow the steps to install the agent in the new Azure Virtual Machine from the [**Exercise 1**](APL2001_M01_L01_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md).
   The differences you need to configure are:
   1. To differentiate the new agent pool and the agent, you can create a new agent pool and name it "eShopOnWebSelfPoolManaged", and then name the agent "eShopOnWebSelfAgentManaged".
   2. During the User account configuration, select "NT AUTHORITY\NETWORK SERVICE" as the account to run the service.
3. Once you have the agent installed, open your agent pool in the Azure DevOps portal and check that the new agent is available.

    ![Screenshot of the new agent in the new agent pool.](media/new-agent-pool.png)

### Exercise 4: Create a new service connection using the Managed Identity and update the CD pipeline

In this exercise, you will create a new service connection using the Managed Identity authentication method. Then, you will update the CD pipeline to use the new service connection.

#### Task 1: Create a new service connection

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the **eShopOnWeb** project and navigate to **Project settings > Service connections**.
3. Click on the **New service connection** button and select **Azure Resource Manager**.
4. Select **Managed Identity** as the **Authentication method**.
5. Fill in the empty fields using the information gathered during previous steps:
    - Subscription Id, Name and Tenant Id (or clientId).
    - In **Service connection name** type **azure subs managed**. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.

6. Click on **Verify** and **Save**.

#### Task 2: Update the CD pipeline

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the **eShopOnWeb** project and navigate to **Pipelines > Pipelines**.
3. Click on the **eshoponweb-cd-webapp-code** pipeline and click on **Edit**.
4. In the variables section, update the **serviceConnection** variable with the name of the service connection you created in the previous task, **azure subs managed**.
5. Update the pipeline agent pool to use the self-hosted agent pool you created in the previous exercise, **eShopOnWebSelfPoolManaged**.

    ```YAML
        variables:
          resource-group: 'rg-eshoponweb'
          location: 'westus'
          templateFile: '.azure/bicep/webapp.bicep'
          subscriptionid: 'YOURSUBSCRIPTION'
          azureserviceconnection: 'azure subs managed'
          webappname: 'eshoponweb-lab'
    
        stages:
        - stage: Deploy
          displayName: Deploy to WebApp
          jobs:
          - job: Deploy
            pool: eShopOnWebSelfPoolManaged
    
    ```

6. Click on **Save**, choose to commit directly to the main branch, or create a new branch.
7. Click on **Save** again.

    > [!NOTE]
    > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

8. Click to **Run** the pipeline, and then click on **Run** again.
9.  Open the pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", click on **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.
10. The deployment may take a few minutes to complete, wait for the pipeline to execute.
11. You should see from the pipeline logs that the pipeline is using the Managed Identity.

    ![Screenshot of the pipeline logs using the Managed Identity.](media/pipeline-logs-managed-identity.png)

After the pipeline finishes, you can go to the Azure portal and check the new App Service resource.

### Exercise 5: Remove the Azure lab resources

1. In the Azure portal, open the created Resource Group and click on **Delete resource group** for all created resources in this lab.

    ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

    > [!WARNING]
    > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

## Review

In this lab, you learned how to dynamically enable configuration and manage feature flags.
