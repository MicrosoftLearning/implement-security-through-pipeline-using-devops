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

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open your organization settings and then Projects under the General section.
3. Click on the **+ New project** button.
4. Enter eShopSecurity as the name for the new team project.
5. Set the project’s visibility to Private.
6. Click in the **Advanced** section to expand it, and then select Git as your version control and Scrum as the work item process.

    ![Screenshot of the new project dialog with the specified settings.](media/new-team-project.png)

7. Click **Create** to create the new project.
8. You can now switch between the different projects by clicking on the Azure DevOps icon in the upper left corner of the Azure DevOps portal.

    ![Screenshot of the Azure DevOps team projects eShopOnWeb and eShopSecurity.](media/azure-devops-projects.png)

You can manage permissions and settings for each project separately by going to the Project settings menu and selecting the appropriate team project. If you have multiple users or teams working on different projects, you can also assign permissions to each project separately.

#### Task 2: Create a new repository and assign project permissions

1. Open the eShopSecurity project.
2. Open the Repos menu and click on the **Initialize** button to initialize the new repository by adding the README.md file.
3. Open the Project settings menu and select repositories under the Repos section.
4. Select the new **eShopSecurity** repository and click on the **Security** tab.
5. Remove the **Inheritance** permission, and deny access to user and groups that should not have access to the repository.
6. Add only a specific group of users, or user that you want to grant access to the repository and run pipelines from the eShopOnWeb project.
7. Click on the search box, enter the name of the group, select it, and then set the permissions you want to allow or deny for the group or user.
8. Deny all permissions for the added user or group, and allow only Read permission.

    ![Screenshot of the repository security settings with allow for read and deny for all other permissions.](media/repository-security.png)

> [!NOTE]
> Ensure that you have the same group in your eShopOnWeb project. This will allow you to run pipelines from the eShopOnWeb project and access the repository in the eShopSecurity project.

### Exercise 2: Configure a pipeline and template structure to support secure pipelines

#### Task 1: (If done, skip) Import and run the CI pipeline

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the eShopOnWeb project in Azure DevOps.
3. Go to **Pipelines > Pipelines**.
4. Click on **New Pipeline** button.
5. Select **Azure Repos Git (Yaml)**.
6. Select the **eShopOnWeb** repository.
7. Select **Existing Azure Pipelines YAML File**.
8. Select the **/.ado/eshoponweb-ci.yml** file then click on **Continue**.
9. Click the **Run** button to run the pipeline.
10. Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-ci** and click on **Save**.

#### Task 2: (If done, skip) Import and run the CD pipeline

1. Go to **Pipelines > Pipelines**.
2. Click on **New pipeline** button.
3. Select **Azure Repos Git (Yaml)**.
4. Select the **eShopOnWeb** repository.
5. Select **Existing Azure Pipelines YAML File**.
6. Select the **/.ado/eshoponweb-cd-webapp-code.yml** file then click on **Continue**.
7. In the YAML pipeline definition, in the variables section, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
   - **az400eshop-NAME**, with a web app name to be deployed with a global unique name, for example, "eshoponweb-lab-secure".
   - **AZ400-EWebShop-NAME** with the name of your preference, for example, "rg-eshoponweb-secure".
8. Click on **Save and Run**, choose to commit directly to the main branch, or create a new branch.
9. Click on **Save and Run** again.

    > [!NOTE]
    > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

10. Open the pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", click on **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.

    ![Screenshot of the permit access from the YAML pipeline.](media/pipeline-deploy-permit-resource.png)

11. The deployment may take a few minutes to complete, wait for the pipeline to execute. The CD definition consists of the following tasks:
      - **Resources**: it is prepared to automatically trigger based on CI pipeline completion. It also downloads the repository for the bicep file.
      - **AzureResourceManagerTemplateDeployment**: Deploys the Azure Web App using bicep template.
12. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-cd-webapp-code** and click on **Save**.

Now you should have two pipelines running in your eShopOnWeb project.

![Screenshot of the successful executed CI/CD pipelines.](media/pipeline-successful-executed.png)

#### Task 3: Move the CD pipeline variables to a YAML template

In this task, you will create a YAML template to store the variables used in the CD pipeline. This will allow you to reuse the template in other pipelines.

1. Go to **Repos > Files**.
2. Expand the **.ado** folder and click on **New file**.
3. Name the file **eshoponweb-secure-variables.yml** and click on **Create**.
4. Add the variables section used in the CD pipeline to the new file. The file should look like the following:

    ```YAML
    variables:
      resource-group: 'rg-eshoponweb-secure'
      location: 'westus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'eshoponweb-lab-secure'

    ```

    > [!IMPORTANT]
    > Replace the values of the variables with the values of your environment (resource group, location, subscription ID, Azure service connection, and web app name).

5. Click on **Commit**, add a comment, and click on **Commit** button.
6. Open the **eshoponweb-cd-webapp-code.yml** pipeline definition, and replace the variables section with the following:

    ```YAML
    variables:
      - template: eshoponweb-secure-variables.yml
    ```

    > [!NOTE]
    > If you are using a different path for the template file, you will need to update the path in the pipeline definition.

7. Click on **Save** and **Run** the pipeline again.

Now you have a YAML template with the variables used in the CD pipeline. You can reuse this template in other pipelines in scenarios where you need to deploy the same resources. Also, your operations team can controls the resource group and location where the resources are deployed and other information in your template values and you don't need to make any changes to your pipeline definition.

#### Task 4: Move the YAML templates to a separate repository and project

In this task, you will move the YAML templates to a separate repository and project.

1. In your eShopSecurity project, go to **Repos > Files**.
2. Create a new file named **eshoponweb-secure-variables.yml**.
3. Copy the content of the file **.ado/eshoponweb-secure-variables.yml** from the eShopOnWeb repository to the new file.
4. Commit the changes.
5. Open the **eshoponweb-cd-webapp-code.yml** pipeline definition from the eShopOnWeb project.
6. Add the following to the resources section:

    ```YAML
    resources:
      repositories:
        - repository: eShopSecurity
          type: git
          name: eShopSecurity/eShopSecurity #name of the project and repository

    ```

7. Replace the variables section with the following:

    ```YAML
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
    ```

    ![Screenshot of the pipeline definition with the new variables and resource sections.](media/pipeline-variables-resource-section.png)

8. Click on **Save** and **Run** the pipeline again. You will see that the pipeline is using the YAML template from the eShopSecurity repository.

    ![Screenshot of the pipeline execution using the YAML template from the eShopSecurity repository.](media/pipeline-execution-using-template.png)

Now you have the YAML templates in a separate repository and project. You can reuse these templates in other pipelines in scenarios where you need to deploy the same resources. Also, your operations team can controls the resource group, location, security and where the resources are deployed and other information in your template values and you don't need to make any changes to your pipeline definition.

### Exercise 2: Remove the Azure lab resources

1. In the Azure portal, open the created Resource Group and click on **Delete resource group** for all created resources in this lab.

    ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

    > [!WARNING]
    > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

## Review

In this lab, you learned how to configure and organize a secure project and repository structure in Azure DevOps. By managing permissions effectively, you can ensure that the right users have access to the resources they need while maintaining the security and integrity of your DevOps pipelines and processes.
