---
lab:
    title: 'Configure a project and repository structure to support secure pipelines'
    module: 'Module 1: Configure a project and repository structure to support secure pipelines'
---

# Configure a project and repository structure to support secure pipelines

In this lab, you will learn how to configure a project and repository structure in Azure DevOps to support secure pipelines. This lab covers best practices for organizing projects and repositories, assigning permissions, and managing secure files.

These exercises take approximately **30** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

### Exercise 1: Configure a secure project structure

In this exercise, you will configure a secure project structure by creating a new project and assigning it project permissions. Separating responsibilities and resources into different projects or repositories with specific permissions supports security.

#### Task 1: Create a new team project

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Open your **organization settings** at the bottom left corner of the portal and then **Projects** under the General section.

1. Select the **New Project** option and use the following settings:

   - name: **eShopSecurity**
   - visibility: **Private**
   - Advanced: Version Control: **Git**
   - Advanced: Work Item Process: **Scrum**

   ![Screenshot of the new project dialog with the specified settings.](media/new-team-project.png)

1. Select **Create** to create the new project.

1. You can now switch between the different projects by clicking on the Azure DevOps icon in the upper left corner of the Azure DevOps portal.

   ![Screenshot of the Azure DevOps team projects eShopOnWeb and eShopSecurity.](media/azure-devops-projects.png)

You can manage permissions and settings for each project separately by going to the Project settings menu and selecting the appropriate team project. If you have multiple users or teams working on different projects, you can also assign permissions to each project separately.

#### Task 2: Create a new repository and assign project permissions

1. Select the organization name in the upper left corner of the Azure DevOps portal and select the new **eShopSecurity** project.

1. Select the **Repos** menu.

1. Select the **Initialize** button to initialize the new repository by adding the README.md file.

1. Open the **Project settings** menu in the lower left corner of the portal and select **Repositories** under the Repos section.

1. Select the new **eShopSecurity** repository and select the **Security** tab.

   > **Note**: Ensure that you select the Security tab in the specific repository only, and not for all repositories in the project. If you select all repositories, you may lose access to other repositories in the project.

1. Remove the Inherit permissions from parent by unchecking the **Inheritance** toggle button.

1. Select the **Contributors** group and select the **Deny** dropdown for all permissions except **Manage permissions** and **Read**. This will prevent all users from the Contributors group from accessing the repository.

   > **Note**: In a real world scenario, you will deny the manage permissions to the Contributors group as well. For this lab, we are allowing the Contributors group to manage permissions to allow you to complete the lab.

1. Select your user under Users and select the **Allow** button to allow all permissions.

   > **Note**: If you don't see your name in the **Users** section, enter your name in the **Search for users or groups** text box and select it in the list of results.

   ![Screenshot of the repository security settings with allow for read and deny for all other permissions.](media/repository-security.png)

1. Your changes will be saved automatically.

Now only the user you assigned permissions and the administrators can access the repository. This is useful when you want to allow specific users to access the repository and run pipelines from the eShopOnWeb project.

### Exercise 2: Configure a pipeline and template structure to support secure pipelines

#### Task 1: Import and run the CI pipeline

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Open the **eShopOnWeb** project in Azure DevOps.

1. Go to **Pipelines > Pipelines**.

1. Select the **Create Pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Existing Azure Pipelines YAML File**.

1. Select the **/.ado/eshoponweb-ci.yml** file then select **Continue**.

1. Select the **Run** button to run the pipeline.

   > **Note**: Your pipeline will take a name based on the project name. You will rename it to easier identify the pipeline.

1. Go to **Pipelines > Pipelines** and select the recently created pipeline. Select the ellipsis and then select **Rename/move** option.

1. Name it **eshoponweb-ci** and select **Save**.

#### Task 2: Import and run the CD pipeline

1. Go to **Pipelines > Pipelines**.

1. Select **New pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Existing Azure Pipelines YAML File**.

1. Select the **/.ado/eshoponweb-cd-webapp-code.yml** file then select **Continue**.

1. In the YAML pipeline definition under the variables section, customize:

   - **Resource Group** named as **AZ400-EWebShop-NAME** with the name of your preference, for example, **rg-eshoponweb-secure**.
   - **Location** with the name of the Azure region you want to deploy your resources, for example, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
   - **az400-webapp-NAME** with a globally unique name of the web app to be deployed, for example, the string **eshoponweb-lab-secure** followed by a random number.

1. Select **Save and Run** and choose to commit directly to the main branch.

1. Select **Save and Run** again.

1. Open the pipeline run. If you see the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", select **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.

   ![Screenshot of the permit access from the YAML pipeline.](media/pipeline-deploy-permit-resource.png)

1. The deployment may take a few minutes to complete, wait for the pipeline to execute. The pipeline is triggered following the completion of the CI pipeline and it includes the following tasks:

   - **AzureResourceManagerTemplateDeployment**: Deploys the Azure App Service web app using bicep template.
   - **AzureRmWebAppDeployment**: Publishes the Web site to the Azure App Service web app.

1. Your pipeline will take a name based on the project name. Let's rename it for identifying the pipeline better.

1. Go to **Pipelines > Pipelines** and select the recently created pipeline. Select the ellipsis and then select **Rename/move** option.

1. Name it **eshoponweb-cd-webapp-code** and select **Save**.

Now you should have two pipelines running in your eShopOnWeb project.

![Screenshot of the successful executed CI/CD pipelines.](media/pipeline-successful-executed.png)

#### Task 3: Move the CD pipeline variables to a YAML template

In this task, you will create a YAML template to store the variables used in the CD pipeline. This will allow you to reuse the template in other pipelines.

1. Go to **Repos** and then **Files**.

1. Expand the **.ado** folder and select  **New file**.

1. Name the file **eshoponweb-secure-variables.yml** and select **Create**.

1. Add the variables section used in the CD pipeline to the new file. The file should look like the following:

   ```yaml
   variables:
     resource-group: 'rg-eshoponweb-secure'
     location: 'southcentralus' #the name of the Azure region you want to deploy your resources
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs' #the name of the service connection to your Azure subscription
     webappname: 'eshoponweb-lab-secure-XXXXXX' #the globally unique name of the web app
   ```

   > **Important**: Replace the values of the variables with the values of your environment (resource group, location, subscription ID, Azure service connection, and web app name).

1. Select **Commit**, in the commit comment text box, enter `[skip ci]`, and then select **Commit**.

   > **Note**: By adding the `[skip ci]` comment to the commit, you will prevent automatic pipeline execution, which, at this point, runs by default following every change to the repo.

1. From the list of files in the repo, open the **eshoponweb-cd-webapp-code.yml** pipeline definition, and replace the variables section with the following:

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml
   ```

1. Select **Commit**, accept the default comment, and then select **Commit** to run the pipeline again.

1. Verify that the pipeline run completed successfully.

Now you have a YAML template with the variables used in the CD pipeline. You can reuse this template in other pipelines in scenarios where you need to deploy the same resources. Also, your operations team can control the resource group and location where the resources are deployed and other information in your template values and you don't need to make any changes to your pipeline definition.

#### Task 4: Move the YAML templates to a separate repository and project

In this task, you will move the YAML templates to a separate repository and project.

1. In your eShopSecurity project, go to **Repos > Files**.

1. Create a new file named **eshoponweb-secure-variables.yml**.

1. Copy the content of the file **.ado/eshoponweb-secure-variables.yml** from the eShopOnWeb repository to the new file.

1. Commit the changes.

1. Open the **eshoponweb-cd-webapp-code.yml** pipeline definition in the eShopOnWeb repo.

1. Add the following to the resources section before the variables section in the pipeline definition:

   ```yaml
     repositories:
       - repository: eShopSecurity
         type: git
         name: eShopSecurity/eShopSecurity #name of the project and repository
   ```

1. Replace the variables section with the following:

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
   ```

   ![Screenshot of the pipeline definition with the new variables and resource sections.](media/pipeline-variables-resource-section.png)

1. Select **Commit**, accept the default comment, and then select **Commit** to run the pipeline again.

1. Navigate to the pipeline run and verify that  the pipeline is using the YAML file from the eShopSecurity repository.

   ![Screenshot of the pipeline execution using the YAML template from the eShopSecurity repository.](media/pipeline-execution-using-template.png)

Now you have the YAML file in a separate repository and project. You can reuse this file in other pipelines in scenarios where you need to deploy the same resources. Also, your operations team can control the resource group, location, security and where the resources are deployed and other information by modifying values in the YAML file and you don't need to make any changes to your pipeline definition.

## Review

In this lab, you learned how to configure and organize a secure project and repository structure in Azure DevOps. By managing permissions effectively, you can ensure that the right users have access to the resources they need while maintaining the security and integrity of your DevOps pipelines and processes.
