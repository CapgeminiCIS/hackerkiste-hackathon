# Setting up your AKS Cluster

## Introduction

You should now have completed the following things:
1. Setup your GitHub account
3. Added the repository secrets to the example code

Next you will get an overview over the example code to setup your first own AKS Cluster on Azure via Terraform. This will only be the most basic infrastructure setup. In a later steps we will alter the content.

If you want to learn more about the concept of a pipeline you can do it using the [https://docs.github.com/en/actions/quickstart](https://docs.github.com/en/actions/quickstart).

# 1. Why We use the Azure Kubernetes Service
We mentioned many times before that you will create an Azure Kubernetes Service (AKS) cluster but what is AKS actually?

Kubernetes is the most commonly used Container Management Platform created by Google. AKS is a managed Kubernetes cluster service on Azure, the cloud provider that we use in this hackathon. If you want to know more about it you should check out these links:

- [What is Kubernets](https://azure.microsoft.com/de-de/topic/what-is-kubernetes/)
- [What is AKS](https://azure.microsoft.com/de-de/services/kubernetes-service/)
- [But why use Container in the first place?](https://www.youtube.com/watch?v=Gjnup-PuquQ)

**What It Is All About**
<br>Modern applications consists of multiple microservices that run individually and as multiple instances. Every microservice can be an app feature (e.g. user management, frontend page, backend login). Each microservice is encapsulated as a container. Containers allow you to run your software in basically every Kubernetes cluster which provides DevOps engineers with all . You do not need to worry about dependencies and the operating systems.

# 2. GitHub and GitHub Actions
GitHub is the leading platform to manage and share code. With GitHub Actions it provides you with a comprehensive way to deploy your code as well. Automating your code deployment makes it more reliable and repeatable.

# 3. Terraform
Terraform is an infrastructure as code (IaC) tool that allows you to build, change, and version infrastructure safely and efficiently. More:
- [Terraform official](https://www.Terraform.io/intro/index.html)
- [Short Video](https://www.youtube.com/watch?v=tomUWcQ0P3k)

# 4. Getting Started
Everyone in your team should be familiar with the overall concept of containers, Terraform and GitHub to continue. If not, share your knowledge and feel free to ask questions.

# 5. Setting up the Pipeline
The first step in creating a pipeline with GitHub Actions would normally be to select a template and start from there.

To do so you would go to actions in your code repository and select an appropriate template or start from scratch with an empty one.

In our example we already created the necessary files. So to say our own template.

Your first task is to go into your repository and look at the following file.

> **Hint:** The formatting of YAML (yml) files is based on spaces and tabs and therefore the following lines should be copied with care. It is advised to use Visual Studio Code to validate the copied file. Find out more on [How to work with Git Locally](./02_SetupGit.md).


`#File: .github/workflows/aks_cluster.yml`
```
name: 'Stage 1 AKS Cluster'
on: [workflow_dispatch]

jobs:
  Terraform:
    name: 'Terraform'
    env:
      ARM_CLIENT_ID: ${{ secrets.AZURE_AD_CLIENT_ID }}
      ARM_CLIENT_SECRET: ${{ secrets.AZURE_AD_CLIENT_SECRET }}
      ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      ARM_TENANT_ID: ${{ secrets.AZURE_AD_TENANT_ID }}
      TF_VAR_uniquename: ${{ secrets.UNIQUE_NAME }}
    runs-on: ubuntu-latest
    environment: production

    defaults:
      run:
        shell: bash
        working-directory: ./stage_1_AKS_Cluster
    outputs:
      aks: ${{ steps.tfout.outputs.aks }}
      rg: ${{ steps.tfout.outputs.rg }}
      acr_name: ${{ steps.tfout.outputs.acr_name }}
      acr_url: ${{ steps.tfout.outputs.acr_url }}
 
```

## Terraform GitHub Actions
The pipeline we are proposing here is using Terraform to create the resources on Azure. [Github Terraform Doc](https://github.com/hashicorp/setup-Terraform)

## Pipeline Name
Next we specify the name of our GitHub action in our example "`name: Stage 1 AKS Cluster`".

## Triggers
The code starts by using "`on: workflow_dispatch`" which means: The Pipeline has to be started Manually.

There are many automatic triggers you can use. To learn more about triggers check this:
https://docs.github.com/en/actions/reference/events-that-trigger-workflows#workflow_dispatch

## Environment Variables
The environment variables are set in the code runners the pipeline uses. In our example these are known variables by Terraform which it uses to authenticate against Azure.

Also we specified a variable that the Terraform code will use internally which is the previous created variable UNIQUE_NAME.

```
    env:
      ARM_CLIENT_ID: ${{ secrets.AZURE_AD_CLIENT_ID }}
      ARM_CLIENT_SECRET: ${{ secrets.AZURE_AD_CLIENT_SECRET }}
      ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      ARM_TENANT_ID: ${{ secrets.AZURE_AD_TENANT_ID }}
      TF_VAR_uniquename: ${{ secrets.UNIQUE_NAME }}
```
## Code Runner
Next we specify what image we expect our job to run on:
"`runs-on: ubuntu-latest`"

```
jobs:
  deploy:
    runs-on: ubuntu-latest
    
```

## Defaults
In our case the Terraform code is located in a subdirectory. We need to define the "`working-directory`" for all upcoming Terraform tasks.

To learn more about the workflow syntax and jobs visit:
https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobs

## Deployment of AppService and WebApp
Next the script manages Terraform with the built in Terraform CLI "`uses: hashicorp/setup-Terraform@v1`".
```
    - name: Checkout
      uses: actions/checkout@v2

    - uses: hashicorp/setup-Terraform@v1

    - run: Terraform init
    
    - name: Terraform fmt
      id: fmt
      run: Terraform fmt -check
      continue-on-error: false

    - name: Terraform Plan
      id: plan
      run: Terraform plan -no-color
```

## Checkout
Checkout gets the branch from Github onto the worker.
As the worker is created from scratch every job needs to get the sources again.

## Terraform CLI
`   - uses: hashicorp/setup-Terraform@v1`
Defines the built-in commands to be available on the worker from GitHub. The CLI documentation can be found here: https://learn.hashicorp.com/tutorials/Terraform/azure-build?in=Terraform/azure-get-started

## Terraform init
Terraform init sets up the current project environment and connects to the Azure Storage Account defined in your "`stage_1_AKS_Cluster/main.tf`"

## Terraform fmt
Terraform fmt allows you to format your code automatically so it matches the expected syntax. It beautyfies your code as well for better readability. 

We set "`continue-on-error: true`" so you get automatic linting and the pipeline doesn't allow properly formatted code.

## Terraform plan
Creates a plan of the changes needed to be done on Azure to accomplish the defined settings in your Terraform code.

## Dependencies and Environments
You can setup concurrent or sequential tasks with pipelines. In our example we made a sequential task by defining defining that the seconds task needs the first:

"`needs: [Terraform]`"

Also we setup two different environments "production" and "development" those are necessary for an approval workflow. More on that later.

## Terraform apply
The final step is to apply the Terraform code which will setup the defined environment on Azure.

## Terraform `main.tf`
Your `/stage_1_AKS_Cluster/main.tf` file contains all the setting for the desired infrastructure on Azure.

Every Terraform project needs a backend to store the state by default a local file will be used but there are many different available backends. In our case we provide you with an Azure Storage Account.

The account is defined by first stating the resource group and storage account name. Inside a Storage Account we also specify an existing container name. The here selected key is the state file Terraform will use. If it does not exist it will be created by the Terraform CLI. 

The name of the key needs to be unique for every participant. *Please note that only lowercase and numbers are allowed.* 

Also we define the version of the AzureRM Provider

```
Terraform {
    required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=2.46.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "hackerkiste-resources"
    storage_account_name = "hackathonTerraform"
    container_name       = "tfstate"
    key                  = "########.tfstate"
  }
}
```

### Storage Account
To learn more about Azure Storage Accounts checkout https://docs.microsoft.com/de-de/azure/storage/common/storage-account-overview

### Backends
To learn more about Terraform Backends checkout https://www.Terraform.io/docs/language/settings/backends/index.html

### State
To learn more about the Terraform State checkout https://www.Terraform.io/docs/language/state/index.html

## Data and Resource
There are different types of definitions we use to define resources in the code.

### Locals
Locals are local variables defined within the Terraform code.

### Data
Used to get available resources on Azure and read out there current configuration for later use. Data is never altered by Terraform as it is an external resource.

### Resource
A Resource is a managed object by Terraform. If Terraform finds a difference in the Terraform state or the actual status of the resource in Azure it will create a plan on how to alter the state so it matches the definitions in your Terraform code again. 

There are mandatory and optional settings for each resource type.

## Resource Group
We define the following resource group as a resource for later use. In our example everyone will need a different resource group name.

Alter your resource group name in the locals.

```
locals {
  name     = var.uniquename
  location = "West Europe"
}
```

```
resource "azurerm_resource_group" "global" {
  name     = local.name
  location = local.location
}
```

Read more about: 
- [Azure Resource Groups](https://docs.microsoft.com/de-de/azure/azure-resource-manager/management/manage-resource-groups-portal)
- [Terraform Resource Group definitions](https://registry.Terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group)

## Defining a AKS
An AKS service is a managed Kubernetes cluster in Azure. 

```
resource "azurerm_kubernetes_cluster" "aks" {
  name                = local.name
  resource_group_name = azurerm_resource_group.global.name
  location            = local.location
  dns_prefix          = local.name
  node_resource_group = format("%s-%s", azurerm_resource_group.global.name, "aks-rg")

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_D2s_v3"
  }

  identity {
    type = "SystemAssigned"
  }
}
```

## ACR
The Azure Container Registry (ACR) host your container images. We already implemented everything you need into the Terraform pipeline. This code connects the cluster to our central ACR and allows the Kubernetes cluster to pull images from it.

Learn more about:

- [Managed Identities](https://docs.microsoft.com/de-de/azure/active-directory/managed-identities-azure-resources/overview)
- [Azure Container Registry](https://azure.microsoft.com/de-de/services/container-registry/)

```
  identity {
    type = "SystemAssigned"
  }
}

# Reference to a externally created Container Registry
data "azurerm_container_registry" "acr" {
  name                = "hackerkisteregistry"
  resource_group_name = "hackerkiste-resources"
}

# Registry Pull permission for the AKS Cluster
resource "azurerm_role_assignment" "acr" {
  scope                = data.azurerm_container_registry.acr.id
  role_definition_name = "AcrPull"
  principal_id         = azurerm_kubernetes_cluster.aks.kubelet_identity.0.object_id
}
```

# 6. Run Your Pipeline
After you set up your secrets and fixed the code in your repository. You can try to run your workflow. To do so go to actions and select the Terraform workflow on the left site.

## Stage 1: AKS Cluster
Now select the created "Stage 1 AKS Cluster" and click on "Run Workflow" on the right side. After a few seconds the pipeline step will display which you can inspect by drilling into it.
<br><img src="./images/runWorkflow.PNG" width="800"/><br>

### Workflow Progress
Wait for your workflow to finish. If the task does not run through you may ask one of us to help you out.

### Checkout Your Resources on Azure
They should be named after your previously selected unique name.

In the [Portal Azure](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) you will find a Resource Group called `<unique_name>` and a Resource Group called `<unique_name>_aks_rg`.

The first one contains the cluster definition the second one all needed resources. As you Kubernetes cluster is fully integrated by Azure it can create resources in you subscriptions as it needs them.

Review your cluster definition and checkout the tabs on the left side. Can you find your already running services inside the cluster?

# Whats Next?
Continue with the [Stage 2 Ingress Controller](05_Stage_2_Ingress.md).