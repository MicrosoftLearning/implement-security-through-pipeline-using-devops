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

1. Use a private browser session to get a new **personal Microsoft Account (MSA)** at [https://account.microsoft.com](https://account.microsoft.com).
2. Using the same browser session, sign up for a free Azure subscription at [https://azure.microsoft.com/free](https://azure.microsoft.com/free).
3. Open a browser and navigate to [https://portal.azure.com](https://portal.azure.com), then search at the top of the Azure portal screen for **Azure DevOps**. In the resulting page, click **Azure DevOps organizations**.
4. Next, click on the link labelled **My Azure DevOps Organizations** or navigate directly to [https://aex.dev.azure.com](https://aex.dev.azure.com).
5. On the **We need a few more details** page, select **Continue**.
6. In the drop-down box on the left, choose **Default Directory**, instead of “Microsoft Account”.
7. If prompted (*"We need a few more details"*), provide your name, e-mail address, and location and click **Continue**.
8. Back at [https://aex.dev.azure.com](https://aex.dev.azure.com) with **Default Directory** selected click the blue button **Create new organization**.
9. Accept the *Terms of Service* by clicking **Continue**.
10. If prompted (*"Almost done"*), leave the name for the Azure DevOps organization at default (it needs to be a globally unique name) and pick a hosting location close to you from the list.
11. Once the newly created organization opens in **Azure DevOps**, click **Organization settings** in the bottom left corner.
12. At the **Organization settings** screen click **Billing** (opening this screen takes a few seconds).
13. Click **Setup billing** and on the right-hand side of the screen select your **Azure Subscription** and click **Save** to link the subscription with the organization.
14. Once the screen shows the linked Azure Subscription ID at the top, change the number of **Paid parallel jobs** for **MS Hosted CI/CD** from 0 to **1**. Then click the **SAVE** button at the bottom.
15. You may **wait at least 3 hours before using the CI/CD capabilities** so that the new settings are reflected in the backend. Otherwise, you will still see the message *"No hosted parallelism has been purchased or granted"*.

## Instructions to create the sample Azure DevOps project (you only have to do this once)

> **Note**: make sure you completed the steps to create your Azure DevOps Organization before continuing with these steps.

To follow all lab instructions, you'll need set up a new Azure DevOps project and create a repository that's based on the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) application.

### Create and configure the team project

First, you'll create an **eShopOnWeb** Azure DevOps project to be used by several labs.

1. Open your browser and navigate to your Azure DevOps organization.
2. Select the **New Project** option and use the following settings:
   - name: **eShopOnWeb**
   - visibility: **Private**
   - Advanced: Version Control: **Git**
   - Advanced: Work Item Process: **Scrum**

3. Click **Create**.

    ![Create Project](media/create-project.png)

### Import eShopOnWeb git repository

Now, you'll import the eShopOnWeb into your git repository.

1. Open your browser and navigate to your Azure DevOps organization.
2. Open the previously created **eShopOnWeb** project.
3. Click on **Repos > Files**, **Import a Repository** and select **Import**.
4. On the **Import a Git Repository** window, paste the following URL https://github.com/MicrosoftLearning/eShopOnWeb and click **Import**:

    ![Import Repository](media/import-repo.png)

5. The repository is organized the following way:
    - **.ado** folder contains Azure DevOps YAML pipelines.
    - **.devcontainer** folder container setup to develop using containers (either locally in VS Code or GitHub Codespaces).
    - **.azure** folder contains Bicep & ARM infrastructure as code templates.
    - **.github** folder container YAML GitHub workflow definitions.
    - **src** folder contains the .NET 6 website used on the lab scenarios.

You have now completed the necessary prerequisite steps to continue with the labs.
