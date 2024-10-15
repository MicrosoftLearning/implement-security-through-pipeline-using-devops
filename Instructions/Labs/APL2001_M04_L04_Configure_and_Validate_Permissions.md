---
lab:
    title: 'Configure and validate permissions'
    module: 'Module 4: Configure and validate permissions'
---

# Configure and validate permissions

In this lab, you'll set up a secure environment that adheres to the principle of least privilege, ensuring that members can access only the resources they need to perform their tasks and minimize potential security risks. This involves configuring and validating user and pipeline permissions and setting up approval and branch checks in Azure DevOps.

These exercises take approximately **20** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).
- Install a self-hosted agent following the lab [Configure agents and agent pools for secure pipelines](APL2001_M02_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md) or the steps in [Install a self-hosted agent](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent).

## Instructions

### Exercise 0: (skip if done) Import and run CI/CD Pipelines

In this exercise, you will import and run the CI/CD pipelines in the Azure DevOps project.

#### Task 1: (skip if done) Import and run the CI pipeline

Let's start by importing the CI pipeline named [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigate to the Azure DevOps portal at `https://aex.dev.azure.com` and open your organization.

1. Open the **eShopOnWeb** project in Azure DevOps.

1. Go to **Pipelines > Pipelines**.

1. Select the **Create Pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Existing Azure Pipelines YAML File**.

1. Select the **/.ado/eshoponweb-ci.yml** file then click on **Continue**.

1. Select the **Run** button to run the pipeline.

   > **Note**: Your pipeline will take a name based on the project name. You will rename it to easier identify the pipeline.

1. Go to **Pipelines > Pipelines** and select the recently created pipeline. Select the ellipsis and then select **Rename/move** option.

1. Name it **eshoponweb-ci** and select **Save**.

#### Task 2: (skip if done) Import and run the CD pipeline

> **Note**: In this task, you will import and run the CD pipeline named [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Go to **Pipelines > Pipelines**.

1. Select **New pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Existing Azure Pipelines YAML File**.

1. Select the **/.ado/eshoponweb-cd-webapp-code.yml** file then select **Continue**.

1. In the YAML pipeline definition, set the variables section to:

   ```yaml
   variables:
     resource-group: 'AZ400-EWebShop-NAME'
     location: 'southcentralus'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs'
     webappname: 'az400-webapp-NAME'
   ```

1. In the variables section, replace the placeholders with the following values:

   - **AZ400-EWebShop-NAME** with the name of your preference, for example, **rg-eshoponweb**.
   - **location** with the name of the Azure region you want to deploy your resources, for example, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
   - **Resource Group** named as **AZ400-EWebShop-NAME** with the name of your preference, for example, **rg-eshoponweb-secure**.

1. Select **Save and Run** and choose to commit directly to the main branch.

1. Select **Save and Run** again.

1. Open the pipeline run. If you receive the message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp", select **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure App Service resource.

   ![Screenshot of the permit access from the YAML pipeline.](media/pipeline-deploy-permit-resource.png)

1. The deployment may take a few minutes to complete, wait for the pipeline to execute. The pipeline is triggered following the completion of the CI pipeline and it includes the following tasks:

   - **AzureResourceManagerTemplateDeployment**: Deploys the Azure App Service web app using bicep template.
   - **AzureRmWebAppDeployment**: Publishes the Web site to the Azure App Service web app.

   > **Note**: In case the deployment fails, navigate to the pipeline run page and select **Rerun failed jobs** to invoke another pipeline run.

   > **Note**: Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better.

1. Go to **Pipelines > Pipelines** and select the recently created pipeline. Select the ellipsis and then select **Rename/move** option.

1. Name it **eshoponweb-cd-webapp-code** and click on **Save**.

### Exercise 1: Configure and validate approval and branch checks

In this exercise, you will configure and validate approval and branch checks for the CD pipeline.

#### Task 1: Create an environment and add approvals and checks

1. In the Azure DevOps portal, from the **eShopOnWeb** project page, select **Pipelines > Environments**.

1. Select **Create environment**.

1. Name the environment **Test**, select **None** as the resource, and select **Create**.

1. In the **Test** environment, select the **Approvals and checks** tab.

1. Select **Approvals**.

1. In the **Approvers** text box, enter your user name.

1. If not enabled, check the box labeled "Allow approvers to approve their own runs."

1. Give the instructions **Approve the deployment to Test** and select **Create**.

   ![Screenshot of the environment approvals with instructions.](media/add-environment-approvals.png)

1. Click on **+ Add new** button, select **Branch control**, and then select **Next**.

1. In the **Allowed branches** field, leave the default and select **Create**. You can add more branches if you want.

   ![Screenshot of the environment branch control with the main branch.](media/add-environment-branch-control.png)

1. Create another environment named **Production** and perform the same steps to add approvals and branch control. To differentiate the environments, add the instructions **Approve the deployment to Production** and set the allowed branches to **refs/heads/main**.

> **Note**: You could add more environments and configure approvals and branch control for them. Additionally, you could configure **Security** to add users or groups to the environment with such roles as *User*, *Creator* or *Reader*.

#### Task 2: Configure the CD pipeline to use the new environment

1. In the Azure DevOps portal, from the **eShopOnWeb** project page, select **Pipelines > Pipelines**.

1. Open the **eshoponweb-cd-webapp-code** pipeline.

1. Select **Edit**.

1. Select the line above the **#download artifacts** comment, up to the **stages:** line in the pipeline YAML file and replace the content with the following code:

   ```yaml
   stages:
   - stage: Test
     displayName: Testing WebApp
     jobs:
     - deployment: Test
       pool: eShopOnWebSelfPool
       environment: Test
       strategy:
         runOnce:
           deploy:
             steps:
             - script: echo Hello world! Testing environments!
   - stage: Deploy
     displayName: Deploy to WebApp
     jobs:
     - deployment: Deploy
       pool: eShopOnWebSelfPool
       environment: Production
       strategy:
         runOnce:
           deploy:
             steps:
             - checkout: self
   ```

   > **Note**: You will need to shift all the lines following the code above six spaces to the right to ensure that YAML indentation rules are satisfied.

   Your pipeline should look like this:

   ![Screenshot of the pipeline with the new deployment.](media/pipeline-add-yaml-deployment.png)

   > [!IMPORTANT]
   > Confirm that the **pool** name is the same as the one you created in the previous lab.

1. Click on **Validate and save**, choose to commit directly to the main branch, and then click on **Save**.

1. Your pipeline will trigger automatically. Open the pipeline run.

   > **Note**: If you receive a message "This pipeline needs permission to access a resource before this run can continue to Testing WebApp" select **View**, **Permit** and **Permit** again.

1. Open the **Testing WebApp** stage of the pipeline and note the message **1 approval needs your review before this run can continue to Testing WebApp**. Select **Review** and select **Approve**.

   ![Screenshot of the pipeline with the Test stage to be approved".](media/pipeline-test-environment-approve.png)

1. Wait for the pipeline to finish, open the pipeline log and check that the **Testing WebApp** stage was executed successfully.

   ![Screenshot of the pipeline log with the Testing WebApp stage executed successfully".](media/pipeline-test-environment-success.png)

1. Back to the pipeline and you will see the stage **Deploy to WebApp** waiting for approval. Select **Review** and **Approve** as you did before for the **Testing WebApp** stage.

   > **Note**: If you receive a message "This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp" select **View**, **Permit** and **Permit** again.

1. Wait for the pipeline to finish and check that the **Deploy to WebApp** stage was executed successfully.

   ![Screenshot of the pipeline with the Deploy to WebApp stage to be approved".](media/pipeline-deploy-environment-success.png)

> **Note**: You should be able to run the pipeline successfully with the approvals and branch checks in both environments, Test and Production.

> [!IMPORTANT]
> Remember to delete the resources created in the Azure portal to avoid unnecessary charges.

## Review

In this lab, you have learned how to set up a secure environment that adheres to the principle of least privilege, ensuring that members can access only the resources they need to perform their tasks and minimize potential security risks. You configured and validated user and pipeline permissions and set up approval and branch checks in Azure DevOps.
