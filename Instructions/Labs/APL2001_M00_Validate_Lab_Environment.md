---
lab:
    title: 'Validate your lab environment'
    module: 'Module 0: Welcome'
---

# Validate your lab environment

In preparation for the labs, it is crucial to have your environment correctly set up. This page will guide you through the setup process, ensuring all prerequisites are met.

- The labs require **Microsoft Edge** or an [Azure DevOps-supported browser.](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Set up an Azure Subscription:** If you don't already have an Azure subscription, create one by following the instructions on this page or visit [https://azure.microsoft.com/free](https://azure.microsoft.com/free) to sign up for a free.

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for the labs, create one by following the instructions on this page, or at [Create an organization or project collection](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
  
- [Git for Windows download page](https://gitforwindows.org/). This will be installed as part of prerequisites for this lab.

- [Visual Studio Code](https://code.visualstudio.com/). This will be installed as part of prerequisites for this lab.

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli). Install the Azure CLI on the self-hosted agent machines.

## Instructions to create an Azure DevOps Organization (you only have to do this once)

> **Note**: Start at step 3, if you do already have a **personal Microsoft Account** setup and an active Azure Subscription linked to that account.

1. Use a private browser session to get a new **personal Microsoft Account (MSA)** at `https://account.microsoft.com`.

1. Using the same browser session, sign up for a free Azure subscription at `https://azure.microsoft.com/free`.

1. Open a browser and navigate to Azure portal at `https://portal.azure.com`, then search at the top of the Azure portal screen for **Azure DevOps**. In the resulting page, click **Azure DevOps organizations**.

1. Next, click on the link labelled **My Azure DevOps Organizations** or navigate directly to `https://aex.dev.azure.com`.

1. On the **We need a few more details** page, select **Continue**.

1. In the drop-down box on the left, choose **Default Directory**, instead of **Microsoft Account**.

1. If prompted (*"We need a few more details"*), provide your name, e-mail address, and location and click **Continue**.

1. Back at `https://aex.dev.azure.com` with **Default Directory** selected click the blue button **Create new organization**.

1. Accept the *Terms of Service* by clicking **Continue**.

1. If prompted (*"Almost done"*), leave the name for the Azure DevOps organization at default (it needs to be a globally unique name) and pick a hosting location close to you from the list.

1. Once the newly created organization opens in **Azure DevOps**, select **Organization settings** in the bottom left corner.

1. At the **Organization settings** screen select **Billing** (opening this screen takes a few seconds).

1. Select **Setup billing** and on the right-hand side of the screen, select your **Azure Subscription** and then select **Save** to link the subscription with the organization.

1. Once the screen shows the linked Azure Subscription ID at the top, change the number of **Paid parallel jobs** for **MS Hosted CI/CD** from 0 to **1**. Then select **SAVE** button at the bottom.

1. You may **wait at least 3 hours before using the CI/CD capabilities** so that the new settings are reflected in the backend. Otherwise, you will still see the message *"No hosted parallelism has been purchased or granted"*.

## Instructions to create and configure the Azure DevOps project (you only have to do this once)

> **Note**: make sure you completed the steps to create your Azure DevOps Organization before continuing with these steps.

To follow all lab instructions, you'll need set up a new Azure DevOps project, create a repository that's based on the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) application, and create a service connection to your Azure subscription.

### Create the team project

First, you'll create an **eShopOnWeb** Azure DevOps project to be used by several labs.

1. Open your browser and navigate to your Azure DevOps organization.

1. Select the **New Project** option and use the following settings:
   - name: **eShopOnWeb**
   - visibility: **Private**
   - Advanced: Version Control: **Git**
   - Advanced: Work Item Process: **Scrum**

1. Select **Create**.

   ![Create Project](media/create-project.png)

### Import eShopOnWeb git repository

Now, you'll import the eShopOnWeb into your git repository.

1. Open your browser and navigate to your Azure DevOps organization.

1. Open the previously created **eShopOnWeb** project.

1. Select the **Repos > Files**, **Import a Repository** and then select **Import**.

1. On the **Import a Git Repository** window, paste the following URL `https://github.com/MicrosoftLearning/eShopOnWeb` and select **Import**:

   ![Import Repository](media/import-repo.png)

1. The repository is organized the following way:

   - **.ado** folder contains Azure DevOps YAML pipelines.
   - **.devcontainer** folder container setup to develop using containers (either locally in VS Code or GitHub Codespaces).
   - **.azure** folder contains Bicep & ARM infrastructure as code templates.
   - **.github** folder container YAML GitHub workflow definitions.
   - **src** folder contains the .NET 8 website used on the lab scenarios.

1. Leave the web browser window open.  

### Create a service connection to access Azure resources

Next, you will create a service connection in Azure DevOps which will allow you to deploy and access resources in your Azure subscription.

1. Start a web browser, navigate to the Azure DevOps portal with the **eShopOnWeb** project open and select **Project settings** in the bottom left corner of the portal.

1. Select the **Service connections** under Pipelines, and then select **Create service connection** button.

   ![Screenshot of the new service connection creation button.](media/new-service-connection.png)

1. On the **New service connection** blade, select **Azure Resource Manager** and **Next** (may need to scroll down).

1. Select **Workload Identity federation (automatic)** and **Next**.

   > **Note**: You can also use **Workload identity federation (manual)** if you prefer to manually configure the service connection. Follow the steps in the [Azure DevOps documentation](https://learn.microsoft.com/azure/devops/pipelines/library/connect-to-azure) to create the service connection manually.

1. Fill in the empty fields using the information:
    - **Subscription**: Select your Azure subscription.
    - **Resource group**: Select the resource group where you want to deploy resources.
    - **Service connection name**: Type **`azure subs`**. This name will be referenced in YAML pipelines to access your Azure subscription.

1. Make sure the **Grant access permission to all pipelines** option is unchecked and select **Save**.

   > **Note:** The **Grant access permission to all pipelines** option is not recommended for production environments. It is only used in this lab to simplify the configuration of the pipeline.

   > **Note**: If you see an error message indicating you don't have the necessary permissions to create a service connection, try again, or configure the service connection manually.

You have now completed the necessary prerequisite steps to continue with the labs.
