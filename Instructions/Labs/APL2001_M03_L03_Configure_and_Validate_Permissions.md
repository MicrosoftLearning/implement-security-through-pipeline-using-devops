---
lab:
    title: 'Configure and validate permissions'
    module: 'Module 3: Configure and validate permissions'
---

# Configure and validate permissions

In this lab, you'll set up a secure environment that adheres to the principle of least privilege, ensuring that members can access only the resources they need to perform their tasks and minimize potential security risks. This involves configuring and validating user and pipeline permissions and setting up approval and branch checks in Azure DevOps.

These exercises take approximately **30** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

### Exercise 1: Import CI pipeline and configure pipeline specific permissions

In this exercise, you will import and run the CI pipeline for the eShopOnWeb application, and configure pipeline specific permissions.

#### Task 1: Import and run the CI pipeline

> [!NOTE]
> Skip the import if already done in another lab.

Start by importing the CI pipeline named [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Open a browser and go to the eShopOnWeb project in Azure DevOps.
2. Go to **Pipelines > Pipelines**.
3. Click on **New Pipeline** button.
4. Select **Azure Repos Git (Yaml)**.
5. Select the **eShopOnWeb** repository.
6. Select **Existing Azure Pipelines YAML File**.
7. Select the **/.ado/eshoponweb-ci.yml** file then click on **Continue**.
8. Click the **Run** button to run the pipeline.
9. Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-ci** and click on **Save**.

### Task 2: Configure and run the pipeline with specific permissions

In this task, you will configure the CI pipeline to run with a specific agent pool, and validate the permissions to run the pipeline. You need to have permissions to edit the pipeline and to add permissions to the agent pool.

1. Go to Project Settings, and select **Agent Pools** under **Pipelines**.
2. Open the **Default** agent pool.
3. Click on **Security** tab.
4. If there is no restriction on the agent pool, click on Restrict permissions button.

    ![Screenshot of the agent pool security tab with no restrictions.](media/agent-pool-security-no-restriction.png)
5. Click on **Add** button and select the **eshoponweb-ci** pipeline to add it to the list of pipelines with access to the agent pool.
6. Back to the **eshoponweb-ci** pipeline, click on **Edit** button.
7. Update the YAML file to use the **Default** agent pool. Replace the **pool** section with the following code:

    ```yaml
    pool: Default

    ```

    > [!NOTE]
    > To run the pipeline with the self-hosted agent, you will need to have the agent running and all the prerequisites installed, for example, Visual Studio to build the solution.

8. Click on **Save**, choose to commit directly to the main branch, or create a new branch.

    > [!NOTE]
    > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

9. Click on **Run** button to run the pipeline.
10. Open the in progress pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Build .Net Core Solution", click on **View**, **Permit** and **Permit** again.

The should be able to run the pipeline successfully.

#### Task 3: Configure the CD pipeline and validate permissions

> [!NOTE]
> Skip the import if already done in another lab.

In this task, you will configure the CD pipeline without adding any permissions to the same agent pool used by the CI pipeline. This will cause the pipeline to fail. Then, you will add the permissions to the agent pool and run the pipeline successfully.

> [!IMPORTANT]
> If you have permissions, you will be able to **Permit** the pipeline to run directly from the executing pipeline. If you don't have permissions, you will need to use another account with administration permissions to enable your pipeline to run using the specific agent as described in the previous Task 2, or to add user permissions to the agent pool.

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

8. Update the YAML file to use the **Default** agent pool. Replace the **pool** section with the following code:

    ```yaml
    pool: Default

    ```

9. Click on **Save** and Run**.
10. Open the pipeline, and you will see the message "This pipeline needs permission to access a resource before this run can continue to Deploy Web App". Click on **View** and there will be no option to **Permit** the pipeline to run.

    ![Screenshot of the pipeline with the error message and no permit button".](media/pipeline-permission-no-permit.png)

#### Task 4: Add permissions to the agent pool

In this task, you will add permissions to the agent pool to allow the CD pipeline to run.

1. Go to Project Settings, and select **Agent Pools** under **Pipelines**.
2. Open the **Default** agent pool.
3. Click on **Security** tab.
4. Under **User permissions** click on **Add** button.
5. Select the user that is running the pipeline and the **User** role, then click on **Add**.

    ![Screenshot of the agent pool security tab adding the user.](media/agent-pool-security-add-user-permissions.png)

6. Back to the **eshoponweb-cd-webapp-code** pipeline, click on **Run** button to run the pipeline.
7. Open the pipeline, and you will see the message "This pipeline needs permission to access a resource before this run can continue to Deploy Web App". Click on **View**, **Permit** and **Permit** again.

You should be able to run the pipeline successfully.

### Exercise 2: Configure and validate approval and branch checks

In this exercise, you will configure and validate approval and branch checks for the CD pipeline.

#### Task 1: Create a new Environment and add approvals and checks

1. Go to **Pipelines > Environments**.
2. Click on **New environment** button.
3. Name the environment **Test**, select **None** as the resource, and click on **Create**.
4. Click on **New environment**, create a new environment **Production**, select **None** as the resource and click on **Create**.
5. Open the **Test** environment, click on ***...*** and select **Approvals and checks**.
6. Click on **Approvals**.
7. Click on **Add approver** and select your user, and if you have another user, add it to validate the approval process.
8. Give the instructions **Please approve the deployment to Test** and click on **Save**.

    ![Screenshot of the environment approvals with instructions.](media/add-environment-approvals.png)
9. Click on **+** button and select **Branch control**.
10. In the **Allowed branches** field, leave the default * and click on **Save**. You can add more branches if you want.

    ![Screenshot of the environment branch control with the main branch.](media/add-environment-branch-control.png)

11. Open the **Production** environment, and perform the same steps to add approvals and branch control. To differentiate the environments, add the instructions **Please approve the deployment to Production** and add the **refs/heads/main** branch to the allowed branches.
12. (Optional) You can add more environments and configure approvals and branch control for them. Additionally, you can configure **Security** to add users or groups to the environment.
    1. Open the **Test** environment, click on ***...*** and select **Security**.
    2. Click on **Add** and select the user that is running the pipeline, and the role *User*, *Creator* or *Reader*.
    3. Click on **Add**.
    4. Click on **Save**.

#### Task 2: Configure the CD pipeline to use the new environment

1. Go to **Pipelines > Pipelines**.
2. Open the **eshoponweb-cd-webapp-code** pipeline.
3. Click on **Edit**.
4. Above the **#download artifacts** comment, add:

    ```yaml
    stages:
   - stage: Test
     displayName: Testing WebApp
     jobs:
     - deployment: Test
       pool:
         vmImage: 'windows-latest'
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
       pool:
         vmImage: 'windows-latest'
       environment: Production
       strategy:
         runOnce:
           deploy:
             steps:

    ```

    Your pipeline should look like this:

    ![Screenshot of the pipeline with the new deployment.](media/pipeline-add-yaml-deployment.png)

5. Click on **Save** and **Run**.
6. Open the pipeline, and you will see the message "This pipeline needs permission to access a resource before this run can continue to Testing WebApp". Click on **View**, **Permit** and **Permit** again.

    ![Screenshot of the pipeline with the permit button to approve the pipeline execution".](media/pipeline-environment-permit.png)
7. Open the "Testing WebApp" stage, and you will see the message "1 approval needs your review before this run can continue to Testing WebApp". Click on **Review** and **Approve**.

    ![Screenshot of the pipeline with the Test stage to be approved".](media/pipeline-test-environment-approve.png)
8. Wait for the pipeline to finish, open the pipeline log and check that the "Testing WebApp" stage was executed successfully.

    ![Screenshot of the pipeline log with the Testing WebApp stage executed successfully".](media/pipeline-test-environment-success.png)
9. Back to the pipeline and you will see the stage "Deploy to WebApp" waiting for approval. Click on **Review** and **Approve** as you did before for the "Testing WebApp" stage.
10. Wait for the pipeline to finish amd check that the "Deploy to WebApp" stage was executed successfully.

    ![Screenshot of the pipeline with the Deploy to WebApp stage to be approved".](media/pipeline-deploy-environment-success.png)

You should be able to run the pipeline successfully with the approvals and branch checks in both environments, Test and Production.

### Exercise 3: Remove the resources used in the lab

1. In the Azure portal, open the created Resource Group and click on **Delete resource group** for all created resources in this lab.

    ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

    > [!WARNING]
    > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

2. Reset the specific permissions you added to the Azure DevOps organization and project in this lab.

## Review

In this lab, you have learned how to set up a secure environment that adheres to the principle of least privilege, ensuring that members can access only the resources they need to perform their tasks and minimize potential security risks. You configured and validated user and pipeline permissions and set up approval and branch checks in Azure DevOps.
