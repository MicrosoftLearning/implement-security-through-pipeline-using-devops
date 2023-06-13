---
lab:
    title: 'Lab: Configure pipelines to securely use variables and parameters'
    module: 'Module 6: Configure pipelines to securely use variables and parameters'
---

# Lab: Configure pipelines to securely use variables and parameters

In this lab, explore the importance of configuring pipelines to use variables and parameters securely in Azure DevOps. This lab covers fundamental concepts and best practices for ensuring that parameters and variables retain their type, identifying and restricting insecure use of parameters and variables, moving parameters into a YAML file that protects their type, limiting variables that can be set at queue time, and validating that mandatory variables are present and set correctly.

These exercises take approximately **30** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

### Exercise 1: Ensure parameters and variable types

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
9. Your pipeline will take a name based on the project name. Rename it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/Remove** option. Name it **eshoponweb-ci-parameters** and click on **Save**.

#### Task 2: Ensure parameters types

1. In your pipeline definition, add a parameters section at the top:

    ```YAML
    parameters:
      - name: buildConfiguration
        type: string
        default: 'Release'
    
    ```

2. Use the parameter in your steps:

    ```YAML
    steps:
    - script: echo Building in ${{ parameters.buildConfiguration }} mode
    
    ```

#### Task 3: Ensure variable types

1. Define a variable with a specified type:

    ```YAML
    variables:
      isDebug: false
    
    ```

2. Use the variable in your steps:

    ```YAML
    steps:
    - script: echo Debug mode is set to $(isDebug)
    
    ```

3. Save and run the pipeline.

### Exercise 2: Identify and restrict insecure use of parameters and variables

#### Task 1: Identify insecure use of parameters and variables

1. In your pipeline definition, look for any use of parameters or variables that might expose sensitive data. This could be logging the value of a parameter or variable, or passing it to a script in an insecure manner.

#### Task 2: Restrict insecure use of parameters and variables

1. Modify your pipeline to securely handle parameters and variables. For example, if you have a password stored in a variable, ensure it is defined as a secret variable:

    ```YAML
    variables:
      dbPassword: 
        value: $(dbPassword)
        isSecret: true
    
    ```

2. Ensure that secret variables are not logged or exposed in any way:

    ```YAML
    steps:
    - script: echo The password is *** 
    
    ```

3. Save and run the pipeline.

### Exercise 3: Move parameters into a YAML file

#### Task 1: Create a YAML file for parameters

1. In your repository, create a new file named parameters.yml.
2. Define your parameters in this file:

    ```YAML
    parameters:
      - name: buildConfiguration
        type: string
        default: 'Release'
    
    ```

#### Task 2: Reference the parameters file in your pipeline

1. In your pipeline definition, replace the parameters section with a reference to the parameters file:

    ```YAML
    parameters:
    - template: parameters.yml
    
    ```

2. Save and run the pipeline.

### Exercise 4: Limit queue time variables

#### Task 1: Define queue time variables

1. In your pipeline definition, define a variable and set allowOverride to false:

    ```YAML
    variables:
      deployEnvironment:
        value: 'Production'
        allowOverride: false
    
    ```

2. Use the variable in your steps:

    ```YAML
    steps:
    - script: echo Deploying to $(deployEnvironment)
    
    ```

3. Save and run the pipeline.

### Exercise 5: Validate mandatory variables

#### Task 1: Define mandatory variables

1. In your pipeline definition, define a variable without a default value:

    ```YAML
    variables:
      deployKey: 
        value: $(deployKey)
    
    ```

2. Use the variable in your steps:

    ```YAML
    steps:
    - script: echo Deploying with key $(deployKey)
    
    ```

3. Save the pipeline but do not run it yet.

#### Task 2: Validate the mandatory variable

1. Before running the pipeline, go to the Variables tab in the pipeline editor.
2. Check that the deployKey variable is set. If not, set it.
3. Run the pipeline. If the deployKey variable is not set, the pipeline should fail.

### Exercise 6: Remove the resources used in the lab

1. In the Azure portal, open the created Resource Group and click on **Delete resource group** for all created resources in this lab.

    ![Screenshot of the delete resource group button.](media/delete-resource-group.png)

    > [!WARNING]
    > Always remember to remove any created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

## Review

In this lab, you explored the importance of configuring pipelines to use variables and parameters securely in Azure DevOps. This lab covered fundamental concepts and best practices for ensuring that parameters and variables retain their type, identifying and restricting insecure use of parameters and variables, moving parameters into a YAML file that protects their type, limiting variables that can be set at queue time, and validating that mandatory variables are present and set correctly.
