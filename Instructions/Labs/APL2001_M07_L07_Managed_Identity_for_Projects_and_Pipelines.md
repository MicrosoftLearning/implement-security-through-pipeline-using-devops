---
lab:
    title: 'Lab: Managed identity for projects and pipelines'
    module: 'Module 7: Manage identity for projects, pipelines, and agents'
---

# Lab: Managed identity for projects and pipelines

Managed identities offer a secure method for controlling access to Azure resources. Azure handles These identities automatically, allowing you to verify access to services compatible with Azure AD authentication. This means you won't need to embed credentials into your code, enhancing security. In Azure DevOps, managed identities can authenticate Azure resources within your pipelines, simplifying access control without compromising security.

In this lab, you'll create a managed identity to use in a service connection and YAML pipelines using Azure DevOps.

This exercise takes approximately **45** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

- Verify that you have a Microsoft account or an Azure AD account with the Contributor or the Owner role in the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Instructions

### Exercise 1: Import and run CI/CD Pipelines

In this exercise, you will import and run the CI pipeline, configure the service connection with your Azure Subscription and then import and run the CD pipeline.

#### Task 1: Import and run the CI pipeline

Let's start by importing the CI pipeline named [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Open a browser and go to the eShopOnWeb project in Azure DevOps.
2. Go to **Pipelines > Pipelines**.
3. Click on **New Pipeline** button.
4. Select **Azure Repos Git (Yaml)**.
5. Select the **eShopOnWeb** repository.
6. Select **Existing Azure Pipelines YAML File**.
7. Select the **/.ado/eshoponweb-ci.yml** file then click on **Continue**.
8. Click the **Run** button to run the pipeline.
9. Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-ci** and click on **Save**.

#### Task 2: Manage the service connection

You can create a connection from Azure Pipelines to external and remote services for executing tasks in a job.

In this task, you will create a service principal by using the Azure CLI, which will allow Azure DevOps to:

- Deploy resources on your azure subscription
- Deploy the eShopOnWeb application

> [!NOTE]
> If you do already have a service principal, you can proceed directly to the next task.

You will need a service principal to deploy  Azure resources from Azure Pipelines.

A service principal is automatically created by Azure Pipeline when you connect to an Azure subscription from inside a pipeline definition or when you create a new service connection from the project settings page (automatic option). You can also manually create the service principal from the portal or using Azure CLI and re-use it across projects.

1. From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has the Owner role in the Azure subscription you will be using in this lab and has the role of the Global Administrator in the Azure AD tenant associated with this subscription.
2. In the Azure portal, click on the **Cloud Shell** icon, located directly to the right of the search textbox at the top of the page.
3. If prompted to select either **Bash** or **PowerShell**, select **Bash**.

   >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**.

4. From the **Bash** prompt, in the **Cloud Shell** pane, run the following commands to retrieve the values of the Azure subscription ID attribute:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Note**: Copy both values to a text file. You will need them later in this lab.

5. From the **Bash** prompt, in the **Cloud Shell** pane, run the following command to create a service principal:

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Note**: The command will generate a JSON output. Copy the output to text file. You will need it later in this lab.

6. Next, from the lab computer, start a web browser, navigate to the Azure DevOps **eShopOnWeb** project. Click on **Project Settings>Service Connections (under Pipelines)** and **New Service Connection**.

7. On the **New service connection** blade, select **Azure Resource Manager** and **Next** (may need to scroll down).

8. The choose **Service principal (manual)** and click on **Next**.

9. Fill in the empty fields using the information gathered during previous steps:
    - Subscription Id and Name
    - Service Principal Id (or clientId), Key (or Password) and TenantId.
    - In **Service connection name** type **azure subs**. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.

10. Click on **Verify and Save**.

#### Task 3: Import and run the CD pipeline

Let's import the CD pipeline named [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Go to **Pipelines>Pipelines**.
2. Click on **New pipeline** button.
3. Select **Azure Repos Git (Yaml)**.
4. Select the **eShopOnWeb** repository.
5. Select **Existing Azure Pipelines YAML File**.
6. Select the **/.ado/eshoponweb-cd-webapp-code.yml** file then click on **Continue**.
7. In the YAML pipeline definition, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
   - **az400eshop-NAME** replace NAME to make it globally unique.
   - **AZ400-EWebShop-NAME** with the resource group name defined before in the lab.

8. Click on **Save and Run** and wait for the pipeline to execute successfully.

    > **Note**: The deployment may take a few minutes to complete.

    The CD definition consists of the following tasks:
    - **Resources**: it is prepared to automatically trigger based on CI pipeline completion. It also downloads the repository for the bicep file.
    - **AzureResourceManagerTemplateDeployment**: Deploys the Azure Web App using bicep template.

9. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines>Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-cd-webapp-code** and click on **Save**.

### Exercise 3: Manage Azure App Configuration

In this exercise, you will create the App Configuration resource in Azure, enable the managed identity and then test the full solution.

> **Note**: This exercise doesn't require any coding skills. The website's code implements already Azure App Configuration functionalities.

If you want to know how to implement this in your application, please take a look at these tutorials: [Use dynamic configuration in an ASP.NET Core app](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) and [Manage feature flags in Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Task 1: Create the App Configuration resource

1. In the Azure Portal, search for the **App Configuration** service
2. Click **Create app configuration** then select:
    - Your Azure Subscription.
    - The Resource Group created previously (it should be named **AZ400-EWebShop-NAME**).
    - The location.
    - A unique like **appcs-NAME-REGION** for example.
    - Select the **Free** pricing tier.
3. Click on **Review + create** then **Create**.
4. After creating the App Configuration service, go to **Overview** and copy/save the value of the **Endpoint**.

#### Task 2: Enable Managed Identity

1. Go to the Web App deployed using the pipeline (it should be named **az400-webapp-NAME**).
2. In the **Settings** section, click on **Identity** then switch status to **On** in the **System Assigned** section, click **save > yes** and wait a few seconds for the operation to finish.
3. Go back to the App Configuration service and click on **Access control** then **Add role assignment**.
4. In the **Role** section, select **App Configuration Data Reader**.
5. In the **Members** section, check **Manage Identity** then select the managed identity of your Web App (they should have the same name).
6. Click on **Review and assign**.

#### Task 3: Configure the Web App

In order to make sure that your website is accessing App Configuration, you need to update its configuration.

1. Go back to your Web App.
2. In the **Settings** section, click on **Configuration**.
3. Add two new application settings:
    - First app setting
        - **Name:** UseAppConfig
        - **Value:** true
    - Second app setting
        - **Name:** AppConfigEndpoint
        - **Value:** *the value you saved/copied previously from App Configuration Endpoint. It should look like https://appcs-NAME-REGION.azconfig.io*

4. Click **Ok** then **Save** and wait for the settings to be updated.
5. Go to **Overview** and click on **Browse**
6. At this step, you will see no changes in the website since the App Configuration doesn't contain any data. This is what you will do in the next tasks.

#### Task 4: Test the Configuration Management

1. In your website, select **Visual Studio** in the **Brand** drop-down list and click on the arrow button (**>**).
2. You will see a message saying *"THERE ARE NO RESULTS THAT MATCH YOUR SEARCH"*. The goal of this Lab is to be able to update that value without updating the website's code or redeploying it.
3. In order to try this, go back to App Configuration.
4. In the **Operations** section, select **Configuration Explorer**.
5. Click on **Create > Key-value** then add:
    - **Key:** eShopWeb:Settings:NoResultsMessage
    - **Value:** *type your custom message*
6. Click **Apply** then go back to your website and refresh the page.
7. You should see your new message instead of the old default value.

Congratulations! In this task, you tested the **Configuration explorer** in Azure App Configuration.

#### Task 5: Test the Feature Flag

Let's continue to test the Feature manager.

1. In order to try this, go back to App Configuration.
2. In the **Operations** section, select **Feature manager**.
3. Click on **Create** then add:
    - **Enable feature flag:** Checked
    - **Feature flag name:** SalesWeekend
4. Click **Apply** then go back to your website and refresh the page.
5. You should see an image with text "ALL T-SHIRTS ON SALE THIS WEEKEND".
6. You can disable this feature in App Configuration and then you would see that the image disappears.

Congratulations! In this task, you tested the **Feature manager** in Azure App Configuration.

### Exercise 4: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisioned in this lab to eliminate unexpected charges.

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisioned in this lab to eliminate unnecessary charges.

1. In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
2. List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

3. Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to dynamically enable configuration and manage feature flags.
