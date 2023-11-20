---
lab:
    title: 'Extend a pipeline to use multiple templates'
    module: 'Module 5: Extend a pipeline to use multiple templates'
---

# Extend a pipeline to use multiple templates

In this lab, explore the importance of extending a pipeline to multiple templates and how to do it using Azure DevOps. This lab covers fundamental concepts and best practices for creating a multi-stage pipeline, creating a variables template, creating a job template, and creating a stage template.

These exercises take approximately **20** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

### Exercise 1: Create a multi-stage YAML pipeline

#### Task 1: Create a multi-stage main YAML pipeline

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Open the **eShopOnWeb** project.

1. Go to **Pipelines > Pipelines**.

1. Select **New Pipeline** button.

1. Select **Azure Repos Git (Yaml)**.

1. Select the **eShopOnWeb** repository.

1. Select **Starter pipeline**.

1. Replace the content of the **azure-pipelines.yml** file with the following code:

    ```YAML
    trigger:
    - main

    pool:
      vmImage: 'windows-latest'

    stages:
    - stage: Dev
      jobs:
      - job: Build
        steps:
        - script: echo Build
    - stage: Test
      jobs:
      - job: Test
        steps:
        - script: echo Test
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy

    ```

1. Select **Save and run**. Choose if you want to commit directly to the main branch or create a new branch. Select **Save and run** button.

   > [!NOTE]
   > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

1. You will see the pipeline running with the three stages (Dev, Test, and Production) and the corresponding jobs. Wait until the pipeline finishes and back to the **Pipelines** page.

    ![Screenshot of the pipeline running with the three stages and the corresponding jobs](media/eshoponweb-pipeline-multi-stage.png)

1. Select **...** (More options) on the right side of the pipeline you just created and select **Rename/move**.

1. Rename the pipeline to **eShopOnWeb-MultiStage-Main** and select **Save**.

#### Task 2: Create a variables template

1. Go to **Repos > Files**.

1. Expand the **.ado** folder and click on **New file**.

1. Name the file **eshoponweb-variables.yml** and click on **Create**.

1. Add the following code to the file:

    ```YAML
    variables:
      resource-group: 'YOUR-RESOURCE-GROUP-NAME'
      location: 'southcentralus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'YOUR-WEB-APP-NAME'

    ```

    > [!IMPORTANT]
    > Replace the values of the variables with the values of your environment (resource group, location, subscription ID, Azure service connection, and web app name).

1. Select **Commit**, add a comment, and select **Commit** button.

#### Task 3: Prepare the pipeline to use templates

1. Go to **Pipelines > Pipelines**.

1. Open the **eShopOnWeb-MultiStage-Main** pipeline.

1. Select **Edit**.

1. Replace the content of the **azure-pipelines.yml** file with the following code:

    ```YAML
    trigger:
    - main
    variables:
    - template: .ado/eshoponweb-variables.yml
    
    stages:
    - stage: Dev
      jobs:
      - template: .ado/eshoponweb-ci.yml
    - stage: Test
      jobs:
      - template: .ado/eshoponweb-cd-webapp-code.yml
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy to Production or Swap

    ```

1. Save the pipeline.

1. Choose if you want to commit directly to the main branch or create a new branch. Select **Save** button.

   > [!NOTE]
   > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

#### Task 4: Updating CI/CD templates

1. Go to **Pipelines > Pipelines**.

1. Edit the **eshoponweb-ci** pipeline.

1. Remove everything above the **jobs** section.

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    # trigger:
    # - main
    
    resources:
      repositories:
        - repository: self
          trigger: none
    
    stages:
    - stage: Build
      displayName: Build .Net Core Solution

    ```

1. Save the pipeline.

1. Go to **Pipelines > Pipelines**.

1. Edit the **eshoponweb-cd-webapp-code** pipeline.

1. Remove everything above the **jobs** section.

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    
    # Trigger CD when CI executed successfully
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true

    variables:
      resource-group: 'rg-eshoponweb'
      location: 'southcentralus'
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: ''
      azureserviceconnection: 'azure subs'
      webappname: 'eshoponweb-lab'
      # webappname: 'webapp-windows-eshop'
    
    stages:
    - stage: Deploy
      displayName: Deploy to WebApp`

    ```

1. Update the **download** step to:

    ```YAML
    - download: current
      artifact: Website
    - download: current
      artifact: Bicep
    ```

1. Save the pipeline.

1. (Optional) Update the production step to deploy your application to another environment, or swap the deployment slots.

#### Task 5: Run the main pipeline

1. Go to **Pipelines > Pipelines**.

1. Open the **eShopOnWeb-MultiStage-Main** pipeline.

1. Select **Run pipeline**.

1. Wait until the pipeline finishes and check the results.

    ![Screenshot of the pipeline running with the three stages and the corresponding jobs](media/multi-stage-completed.png)

### Exercise 2: Remove the Azure lab resources

1. In the Azure portal, open the created Resource Group and select **Delete resource group** for all created resources in this lab.

    ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

    > [!WARNING]
    > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

## Review

In this lab, you learned how to extend a pipeline into multiple templates and how to do it using Azure DevOps. This lab covered fundamental concepts and best practices for creating a multi-stage pipeline, creating a variables template, creating a job template, and creating a stage template.
