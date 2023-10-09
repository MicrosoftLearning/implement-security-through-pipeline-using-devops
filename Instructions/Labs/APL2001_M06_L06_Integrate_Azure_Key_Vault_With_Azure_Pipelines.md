---
lab:
    title: 'Integrate Azure Key Vault with Azure Pipelines'
    module: 'Module 6: Configure secure access to Azure Repos from pipelines'
---

# Integrate Azure Key Vault with Azure Pipelines

Azure Key Vault provides secure storage and management of sensitive data, such as keys, passwords, and certificates. Azure Key Vault includes support for hardware security modules and a range of encryption algorithms and key lengths. By using Azure Key Vault, you can minimize the possibility of disclosing sensitive data through source code, a common mistake developers make. Access to Azure Key Vault requires proper authentication and authorization, supporting fine-grained permissions to its content.

These exercises take approximately **40** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

In this lab, you will see how you can integrate Azure Key Vault with Azure Pipelines by using the following steps:

- Create an Azure Key vault to store an ACR password as a secret.
- Create an Azure Service Principal to access Azure Key Vault's secrets.
- Configure permissions to allow the Service Principal to read the secret.
- Configure the pipeline to retrieve the password from the Azure Key Vault and pass it on to subsequent tasks.

### Exercise 1: Setup CI pipeline to build eShopOnWeb container

Setup CI YAML pipeline for:

- Creating an Azure Container Registry to keep the container images
- Using Docker Compose to build and push **eshoppublicapi** and **eshopwebmvc** container images. Only **eshopwebmvc** container will be deployed.

#### Task 1: Create a service principal

In this task, you will create a Service Principal by using the Azure CLI, which will allow Azure DevOps to:

- Deploy resources on your Azure subscription.
- Have read access on the later created Key Vault secrets.

You will need a Service Principal to deploy Azure resources from Azure Pipelines. Since you'll retrieve secrets in a pipeline, you need to grant permission to the service when we create the Azure Key Vault.

Azure Pipelines automatically creates a Service Principal when you connect to an Azure subscription from inside a pipeline definition or when you create a new Service Connection from the project settings page (automatic option). You can also manually create the Service Principal from the portal or use Azure CLI and reuse it across projects.

1. Start a web browser, navigate to the Azure Portal at `https://portal.azure.com`, and sign in with the user account that has the Owner role in the Azure subscription you will be using in this lab and has the role of the Global Administrator in the Azure AD tenant associated with this subscription.

1. In the Azure portal, click on the **Cloud Shell** icon, located directly to the right of the search textbox at the top of the page.

1. If prompted to select either **Bash** or **PowerShell**, select **Bash**.

   > [!NOTE]
   > If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**.

1. From the **Bash** prompt, in the **Cloud Shell** pane, run the following commands to retrieve the values of the Azure subscription ID and subscription name attributes:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > [!NOTE]
    > Copy both values to a text file. You will need them later in this lab.

1. From the **Bash** prompt, in the **Cloud Shell** pane, run the following command to create a Service Principal (replace the **myServicePrincipalName** with any unique string of characters consisting of letters and digits) and **mySubscriptionID** with your Azure subscriptionId :

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > [!NOTE]
    > The command will generate a JSON output. Copy the output to text file. You will need it later in this lab.

1. Next, navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Navigate to the Azure DevOps **eShopOnWeb** project. Select **Project Settings > Service Connections (under Pipelines)** and **New Service Connection**.

    ![Screenshot of the new service connection creation button.](media/new-service-connection.png)

1. On the **New service connection** blade, select **Azure Resource Manager** and **Next** (may need to scroll down).

1. Then choose **Service Principal (manual)** and select **Next**.

1. Fill in the empty fields using the information gathered during previous steps:
    - Subscription Id and Name.
    - Service Principal Id (or clientId/AppId), Service Principal Key (or Password) and TenantId.
    - In **Service connection name** type **azure subs**. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.

        ![Screenshot of the Azure service connection configuration.](media/azure-service-connection.png)

1. Do not check **Grant access permission to all pipelines**. Select **Verify and Save**.

    > [!NOTE]
    > The **Grant access permission to all pipelines** option is not recommended for production environments. It is only used in this lab to simplify the configuration of the pipeline.

#### Task 2: Setup and run CI pipeline

In this task, you will import an existing CI YAML pipeline definition, modify and run it. It will create a new Azure Container Registry (ACR) and build/publish the eShopOnWeb container images.

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Navigate to the Azure DevOps **eShopOnWeb** project. Go to **Pipelines > Pipelines** and click on **Create Pipeline** (or **New pipeline**).

1. On the **Where is your code?** window, select **Azure Repos Git (YAML)** and select the **eShopOnWeb** repository.

1. On the **Configure** section, choose **Existing Azure Pipelines YAML file**. Provide the following path **/.ado/eshoponweb-ci-dockercompose.yml** and select **Continue**.

    ![Screenshot of the Pipeline selection from the YAML file.](media/select-ci-container-compose.png)

1. In the YAML pipeline definition, in the variables section, customize your Resource Group name by replacing **AZ400-EWebShop-NAME** by the name of your preference, for example, **rg-eshoponweb**, replace **YOUR-SUBSCRIPTION-ID** with your own Azure subscriptionId, and choose the location of your preference nearest to your location, for example **southcentralus**.

1. (Optional) You can use the self-hosted agent created in the previous lab updating the pool name currently set to the Microsoft-hosted agent to the name of the agent pool you created, **eShopOnWebSelfPool**.

    Instead of:

    ```YAML
      - job: Build
        pool:
          vmImage: ubuntu-latest
    
    ```

    Use:

    ```YAML
      - job: Build
        pool: eShopOnWebSelfPool
    
    ```

    > [!NOTE]
    > To run the pipeline with the self-hosted agent, you will need to have the agent running and all the prerequisites installed, for example, Visual Studio to build the solution. If you do not have the prerequisites installed, you can use the Microsoft-hosted agent.

1. Select **Save and Run**, choose to commit directly to the main branch, or create a new branch.

1. Select **Save and Run** again.

    > [!NOTE]
    > If you choose to create a new branch, you will need to create a pull request to merge the changes to the main branch.

1. Open the pipeline. If you see the message "This pipeline needs permission to access a resource before this run can continue to Create ACR for images", select **View**, **Permit** and **Permit** again. This is needed to allow the pipeline to create the Azure Container Registry (ACR) resource.

    ![Screenshot of the permit access from the YAML pipeline.](media/pipeline-permit-resource.png)

1. The build may take a few minutes to complete, wait for the pipeline to execute. The build definition consists of the following tasks:
      - **AzureResourceManagerTemplateDeployment** uses **bicep** to deploy an Azure Container Registry.
      - **PowerShell** task take the bicep output (acr login server) and creates pipeline variable.
      - **DockerCompose** task builds and pushes the container images for eShopOnWeb to the Azure Container Registry .

1. Your pipeline will take a name based on the Project name. Rename it to identify the pipeline better.

1. Go to **Pipelines > Pipelines** on the recently created pipeline, mouse over the executed pipeline, and select the ellipsis and **Rename/Remove** option.

1. Name it **eshoponweb-ci-dockercompose** and select **Save**.

1. Once the execution is finished, on the Azure Portal, open the previously defined Resource Group, and you should find an Azure Container Registry (ACR) with the created container images **eshoppublicapi** and **eshopwebmvc**. You will only use **eshopwebmvc** in the deploy phase.

    ![Screenshot of the container images in ACR from the Azure Portal.](media/azure-container-registry.png)

1. Select **Access Keys** and copy the **password** value, it will be used in the following task, as we will keep it as a secret in Azure Key Vault.

    ![Screenshot of the ACR password from the Access Keys setting.](media/acr-password.png)

#### Task 2: Create an Azure Key Vault

In this task, you will create an Azure Key vault by using the Azure portal.

For this lab scenario, we will have a Azure Container Instance (ACI) that pull and runs a container image stored in Azure Container Registry (ACR). We intend to store the password for the ACR as a secret in the key vault.

1. In the Azure portal, in the **Search resources, services, and docs** text box, type **Key vault** and press the **Enter** key.

1. Select **Key vault** blade, click on **Create > Key Vault**.

1. On the **Basics** tab of the **Create key vault** blade, specify the following settings and click on **Next**:

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | the resource group name **rg-eshoponweb** |
    | Key vault name | any unique valid name, like **ewebshop-kv-NAME** (replace NAME) |
    | Region | an Azure region close to the location of your lab environment |
    | Pricing tier | **Standard** |
    | Days to retain deleted vaults | **7** |
    | Purge protection | **Disable purge protection** |

1. On the **Access policy** tab of the **Create key vault** blade, on the **Access Policy** section, select **+ Create** to setup a new policy.

    > **Note**: You need to secure access to your key vaults by allowing only authorized applications and users. To access the data from the vault, you will need to provide read (Get/List) permissions to the previously created service principal that you will be using for authentication in the pipeline.

    - On the **Permission** blade, check **Get** and **List** permissions below **Secret Permission**. Select **Next**.
    - on the **Principal** blade, search for the **previously created Service Principal**, either using the Id or Name given. Select **Next** and **Next** again.
    - On the **Review + create** blade, select **Create**

1. Back on the **Create a Key Vault** blade, select **Review + Create > Create**

    > [!NOTE]
    > Wait for the Azure Key vault to be provisioned. This should take less than 1 minute.

1. On the **Your deployment is complete** blade, select **Go to resource**.

1. On the Azure Key vault blade, in the vertical menu on the left side of the blade, in the **Objects** section, select **Secrets**.

1. On the **Secrets** blade, select **Generate/Import**.

1. On the **Create a secret** blade, specify the following settings and select **Create** (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Upload options | **Manual** |
    | Name | **acr-secret** |
    | Value | ACR access password copied in previous task |

#### Task 3: Create a Variable Group connected to Azure Key Vault

In this task, you will create a Variable Group in Azure DevOps that will retrieve the ACR password secret from Key Vault using the Service Connection (Service Principal)

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Navigate to the Azure DevOps project **eShopOnWeb**.

1. In the vertical navigational pane of the of the Azure DevOps portal, select **Pipelines > Library**. Select **+ Variable Group**.

1. On the **New variable group** blade, specify the following settings:

    | Setting | Value |
    | --- | --- |
    | Variable Group Name | **eshopweb-vg** |
    | Link secrets from Azure KV ... | **enable** |
    | Azure subscription | **Available Azure service connection > Azure subs** |
    | Key vault name | Your key vault name|

1. Under **Variables**, select **+ Add** and select the **acr-secret** secret. Select **OK**.

1. Select **Save**.

    ![Screenshot of the variable group creation.](media/vg-create.png)

#### Task 4: Setup CD pipeline to deploy container in Azure Container Instance(ACI)

In this task, you will import a CD pipeline, customize it and run it for deploying the container image created before in a Azure Container Instance.

1. Navigate to the Azure DevOps portal at `https://dev.azure.com` and open your organization.

1. Navigate to the Azure DevOps **eShopOnWeb** project. Go to **Pipelines > Pipelines** and select **New Pipeline**.

1. On the **Where is your code?** window, select **Azure Repos Git (YAML)** and select the **eShopOnWeb** repository.

1. On the **Configure** section, choose **Existing Azure Pipelines YAML file**. Provide the following path **/.ado/eshoponweb-cd-aci.yml** and select **Continue**.

1. In the YAML pipeline definition, customize:

    - **YOUR-SUBSCRIPTION-ID** with your Azure subscription id.
    - **az400eshop-NAME** replace NAME to make it globally unique.
    - **YOUR-ACR.azurecr.io** and **ACR-USERNAME** with your ACR login server (both need the ACR name, can be reviewed on the ACR > Access Keys).
    - **rg-eshoponweb** with the resource group name defined before in the lab.

1. Select **Save and Run** and wait for the pipeline to execute successfully.

    > **Note**: The deployment may take a few minutes to complete. The CD definition consists of the following tasks:
    - **Resources** : it is prepared to automatically trigger based on CI pipeline completion. It also download the repository for the bicep file.
    - **Variables (for Deploy stage)** connects to the variable group to consume the Azure Key Vault secret **acr-secret**
    - **AzureResourceManagerTemplateDeployment** deploys the Azure Container Instance (ACI) using bicep template and provides the ACR login parameters to allow ACI to download the previously created container image from Azure Container Registry (ACR).

1. Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better.

1. Go to **Pipelines > Pipelines**, select the recently created pipeline, select the ellipsis and then select **Rename/Remove** option.

1. Name it **eshoponweb-cd-aci** and select **Save**.

### Exercise 2: Remove the Azure lab resources

1. In the Azure portal, open the created Resource Group and click on **Delete resource group**.

    ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

    > [!WARNING]
    > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

## Review

In this lab, you integrated Azure Key Vault with an Azure DevOps pipeline by using the following steps:

- Created an Azure service principal to provide access to Azure Key vault's secrets and authenticate Azure deployment from Azure DevOps.
- Run 2 YAML pipelines imported from a Git repository.
- Configured pipeline to retrieve the password from the Azure Key vault using ADO Variable Group and use it on subsequent tasks.
