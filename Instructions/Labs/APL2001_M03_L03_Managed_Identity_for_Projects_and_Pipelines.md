---
lab:
    title: 'Managed identity for projects and pipelines'
    module: 'Module 3: Manage identity for projects, pipelines, and agents'
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

#### Task 1: Import and run the CI pipeline

Let's start by importing the CI pipeline named [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Open the **eShopOnWeb** project.

1. Go to **Pipelines > Pipelines**.

1. Select **Create Pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Existing Azure Pipelines YAML File**.

1. Select the **/.ado/eshoponweb-ci.yml** file then click on **Continue**.

1. Select the **Run** button to run the pipeline.

   > [!NOTE]
   > Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better.

1. Go to **Pipelines > Pipelines**, select the recently created pipeline, select the ellipsis and then select **Rename/move** option.

1. Name it **eshoponweb-ci** and select **Save**.

> [!NOTE]
> Before you proceed, verify that you already have a service connection to your Azure subscription named **azure subs**. If not, rerun exercise 2, task 2 of the previous lab of this course **Configure a project and repository structure to support secure pipelines**.

> [!NOTE]
> You will also need the value of your subscription ID, which you retrieved in exercise 2, task 2 of the previous lab of this course. If you haven't recorded it, refer to that task for the instructions on getting this accomplished. 

#### Task 2: Import and run the CD pipeline

> [!NOTE]
> In this task, you will import and run the CD pipeline named [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. On the **Pipelines** pane of the **eShopOnWeb** project, select the **New pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Existing Azure Pipelines YAML File**.

1. Select the **/.ado/eshoponweb-cd-webapp-code.yml** file then select **Continue**.

1. In the YAML pipeline definition, set the variables section to:

   ```yaml
   variables:
     resource-group: 'AZ400-EWebShop-NAME'
     location: 'westeurope'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs'
     webappname: 'az400-webapp-NAME'
   ```

1. In the variables section, replace the placeholders with the following values:

   - **AZ400-EWebShop-NAME** with the name of your preference, for example, **rg-eshoponweb**.
   - **location** with the name of the Azure region you want to deploy your resources, for example, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
   - **az400-webapp-NAME**, with a globally unique name of the web app to be deployed, for example, the string **eshoponweb-lab-id-** followed by a random six-digit number. 

1. Select **Save and Run** and choose to commit directly to the main branch.

1. Select **Save and Run** again.

1. Open the pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", selet **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.

   ![Screenshot of the permit access from the YAML pipeline.](media/pipeline-deploy-permit-resource.png)

1. The deployment may take a few minutes to complete, wait for the pipeline to execute. The CD definition consists of the following tasks:

   - **AzureResourceManagerTemplateDeployment**: Deploys the Azure App Service web ppp using bicep template.
   - **AzureRmWebAppDeployment**: Publishes the Web site to the Azure App Service web app.

> [!NOTE]
> In case the deployment fails, navigate to the pipeline run page and select **Rerun failed jobs** to invoke another pipeline run.

1. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better.

1. Go to **Pipelines > Pipelines**, select the recently created pipeline, select the ellipsis and then select **Rename/move** option.

1. Name it **eshoponweb-cd-webapp-code** and then select **Save**.

### Exercise 2: Create a managed identity for the service connection

In this exercise, you will create a managed identity and then create a new service connection to use it in the CI/CD pipelines.

#### Task 1: Create a managed identity

1. In your browser, open the Azure Portal at `https://portal.azure.com`.

1. In the **Search resources, services and docs (G+/)** box, type **Managed Identities** and select it from the dropdown list.

    ![Screenshot of the Managed Identities option in the Azure Portal.](media/managed-identities.png)

1. On the **Managed Identities** page, select the **+ Create** button.

1. On the **Create Managed Identity** pane, perform the following actions:

   - Ensure that the **Subscription** drop-down list includes the name of your Azure subscription.
   - Select **Create new** below the **Resource Group** drop-down list and create a new resource group named **rg-eshoponweb-resources**.
   - Set the **Region** to the same Azure region you chose in the previous exercise of this lab.
   - Enter a name you want to assign to the managed identity in the **Name** textbox (for example, **eshoponweb-mi**).

    ![Screenshot of the create Managed Identity pane.](media/create-managed-identity.png)

1. Select the **Review + create** button, then select **Create**.

#### Task 2: Assign permissions to the Managed Identity

Next, you need to assign the Managed Identity permissions to the resource group and app services.

1. In the Azure portal, navigate to the new Managed Identity you created earlier.

1. Select the **Azure role assignments** tab from let side menu.

1. Select the **Add role assignment** button, and perform the following actions:

   | Setting | Action |
   | -- | -- |
   | **Scope** drop-down list | Select **Resource Group**. |
   | **Subscription** drop-down list | Select your Azure subscription. |
   | **Resource group** drop-down list | Select the resource group where you deployed the web app in the previous exercise of this lab (**rg-eshoponweb**). |
   | **Role** drop-down list | Select the **Contributor** role. |

1. Select the **Save** button.

    ![Screenshot of the add role assignment pane.](media/add-role-assignment.png)

### Exercise 3: Create a new Azure Virtual Machine using self-hosted agent and the Managed Identity and update the CI pipeline

In this exercise, you will create a new Azure Virtual Machine using the self-hosted agent and the Managed Identity you created in the previous exercise. Then, you will update the CI pipeline to use the new Azure Virtual Machine.






1. In the Azure portal, navigate to the page of the newly dpeloyed virtual machine. 

1. In the vertical menu on the left side, in the **Security** section, select **Identity**.

1. On the **eshoponweb-vm /| Identity** page, on the **System assigned** tab, select **Off**, select **Save**, and then select **Yes**.

   > [!NOTE]
   > You could use a system assigned managed identity in this case as well, but we'll leverage the user assigned managed identity you created earlier in this lab.

1. On the **eshoponweb-vm /| Identity** page, select the **User assigned** tab and then select **+ Add**.

1. In the **Add user assigned managed identity** pane, select the user assigned managed identity you created earlier in this lab (**eshopweb-mi**) and then select **Add**.

#### Task 2: Install the self-hosted agent on the Azure Virtual Machine



2. From the Azure VM, follow the steps to install the agent in the new Azure Virtual Machine from the [Exercise 1 of the lab Configure agents and agent pools for secure pipelines](APL2001_M03_L03_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md). When following the instructions, account for the following changes:

   - Name the agent pool **eShopOnWebSelfPoolManaged** (instead of **eShopOnWebSelfPool**) in Task 1 step 5.
   - Name the agent **eShopOnWebSelfAgentManaged** (instead of **eShopOnWebSelfAgent**) in Task 4, step 3.
   - Select **NT AUTHORITY\NETWORK SERVICE** as the account to run the service during during the User account configuration in Task 4, step 3.

3. Once you have the agent installed, open your agent pool in the Azure DevOps portal and check that the new agent is available.

    ![Screenshot of the new agent in the new agent pool.](media/new-agent-pool.png)

### Exercise 4: Create a new service connection using the Managed Identity and update the CD pipeline

In this exercise, you will create a new service connection using the Managed Identity authentication method. Then, you will update the CD pipeline to use the new service connection.

#### Task 1: Create a new service connection

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Open the **eShopOnWeb** project and navigate to **Project settings > Service connections**.

1. Select the **New service connection** button and select **Azure Resource Manager**.

1. Select **Managed Identity** as the **Authentication method**.

1. Fill in the empty fields using the information gathered during previous steps:

    - Subscription Id, Name and Tenant Id (or clientId).
    - In **Service connection name** type **azure subs managed**. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.

1. Select **Verify** and **Save**.

#### Task 2: Update the CD pipeline

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Open the **eShopOnWeb** project and navigate to **Pipelines > Pipelines**.

1. Select the **eshoponweb-cd-webapp-code** pipeline and select **Edit**.

1. In the variables section, update the **serviceConnection** variable with the name of the service connection you created in the previous task, **azure subs managed**.

   ```yaml
         azureserviceconnection: 'azure subs managed'
   ```

1. In the **jobs** subsection of the **stages** section, update the value of the **pool** property to reference the self-hosted agent pool you created in the previous exercise, **eShopOnWebSelfPoolManaged**, so it has the following format:

   ```yaml
         jobs:
         - job: Deploy
           pool: eShopOnWebSelfPoolManaged
           steps:
           #download artifacts
           - download: eshoponweb-ci
   ```

1. Select **Save**, choose to commit directly to the main branch, or create a new branch.

1. Select **Save** again.

   > [!NOTE]
   > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

1. Select to **Run** the pipeline, and then click on **Run** again.

1. Open the pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", select on **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.

1. The deployment may take a few minutes to complete, wait for the pipeline to execute.

1. You should see from the pipeline logs that the pipeline is using the Managed Identity.

   ![Screenshot of the pipeline logs using the Managed Identity.](media/pipeline-logs-managed-identity.png)

After the pipeline finishes, you can go to the Azure portal and check the new App Service resource.

### Exercise 5: Remove the Azure lab resources

1. In the Azure portal, open the created Resource Group and select **Delete resource group** for all created resources in this lab.

   ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

   > [!WARNING]
   > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

## Review

In this lab, you learned how to dynamically enable configuration and manage feature flags.
