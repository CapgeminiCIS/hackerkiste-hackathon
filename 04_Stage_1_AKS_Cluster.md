# Setting up your AKS Cluster

## Introduction

You should now have Completed the Following things:
1. Setup your GitHub Account
2. Setup the Example Code in your Account
3. Added the Repository Secrets to the Example Code

Next you will get an Overview over the Example Code to Setup your first own AKS Cluster on Azure via Terraform. This will only be the most basic Infrastructure Setup. In a later Steps we will Alter the Content.

If you want to learn more about the concept of a pipeline you can do it here:

[https://docs.github.com/en/actions/quickstart](https://docs.github.com/en/actions/quickstart)

# 1. AKS?

We mentioned many time before that you will create an AKS Cluster but what is AKS actually?
AKS stands for Azure Kubernetes Cluster and is managed Kubernetes Cluster on Azure. 
Kubernetes is the most Commonly used Container Management Platform created by Google.
If you want to know more about it you should check out these links

[What is Kubernets](https://azure.microsoft.com/de-de/topic/what-is-kubernetes/)

[What is AKS](https://azure.microsoft.com/de-de/services/kubernetes-service/)

[But why use Container in the first place?](https://www.youtube.com/watch?v=Gjnup-PuquQ)

### What it all boils down to is:
Containers allow you to run your Software everywhere. You don't need to worry about Dependencies and the Operating systems.

# 2. GitHub and GitHub Actions

GitHub is the Leading Platform to Manage and Share Code and with the Introduction of GitHub Actions it provides you with a comprehensive way to Deploy your Code as well.

Automating your Code Deployment makes it more reliable and repeatable.

# 3. Terraform

Terraform is an infrastructure as code (IaC) tool that allows you to build, change, and version infrastructure safely and efficiently. 

https://www.terraform.io/intro/index.html

[Short Video](https://www.youtube.com/watch?v=tomUWcQ0P3k)

# 4. Getting Started

Everyone in your group should be familiar with the overall Concept of Containers, Terraform and GitHub to continue. If not try to share your Knowledge about it and ask Questions about it

# 5. Setting up the Pipeline

The first Step in Creating a Pipeline with GitHub Actions would normally be to Select a Template and Start from there.

To do so you would go to Actions in your Code Repository and select an appropriate Template or start from Scratch with an empty one.

In our Example we already Created the necessary Files. So to say our own Template.

Your first Task is to go into your Repository and look at the Following file.

>_Warning: The formatting of YAML (yml) files is based on spaces and tabs and therefore the following lines should be copied with care.
> It is advised to use Visual Studio Code to validate the copied file._
> <br> [How to work with Git Locally](./02_SetupGit.md)


`#File: .github/workflows/aks_cluster.yml`
```
name: 'Stage 1 AKS Cluster'
on: [workflow_dispatch]

jobs:
  terraform:
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

The Pipeline we are Proposing here is using Terraform to create the Resources on Azure.

Github Terraform Doc: 
<br> https://github.com/hashicorp/setup-terraform

## Pipeline Name

Next we Specify the Name of our GitHub Action in our Example "`name: Stage 1 AKS Cluster`".

## Triggers

The Code Starts by using "`on: workflow_dispatch`" which means one of the Triggers to Start this Pipeline is to do it Manually.

There are many Automatic triggers you can use, to learn more about Triggers check this:
https://docs.github.com/en/actions/reference/events-that-trigger-workflows#workflow_dispatch

## Environment Variables:

The Environment Variables are set in the Code Runners the Pipeline uses. In our Example these are known Variables by Terraform which it uses to Authenticate against Azure.

Also we specified a Variable that the Terraform Code will use internally which is the previous created Variable UNIQUE_NAME.

```
    env:
      ARM_CLIENT_ID: ${{ secrets.AZURE_AD_CLIENT_ID }}
      ARM_CLIENT_SECRET: ${{ secrets.AZURE_AD_CLIENT_SECRET }}
      ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      ARM_TENANT_ID: ${{ secrets.AZURE_AD_TENANT_ID }}
      TF_VAR_uniquename: ${{ secrets.UNIQUE_NAME }}
```
## Code runner

After we define the Triggers and the Name of the workflow we need to Specify its "`jobs`".
In our Example we split the Pipeline into two Jobs "`terraform`" and "`terraformapply`".

This is done to allow for manual Approval. More on that later.

Next we Specify what image we Expect our job to run on:
"`runs-on: ubuntu-latest`"

```
jobs:

  deploy:
    runs-on: ubuntu-latest
    
```

## Defaults

In Our Case the terraform Code is Located in a Subdirectory why we need to define the "`working-directory`" for all upcoming Terraform Tasks. 

To learn more about the Workflow Syntax and Jobs visit:
https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobs

## Deployment of AppService and WebApp

Next the Script manages Terraform with the Build in Terraform CLI "`uses: hashicorp/setup-terraform@v1`".
```
    - name: Checkout
      uses: actions/checkout@v2

    - uses: hashicorp/setup-terraform@v1

    - run: terraform init
    
    - name: Terraform fmt
      id: fmt
      run: terraform fmt -check
      continue-on-error: false

    - name: Terraform Plan
      id: plan
      run: terraform plan -no-color
```

## Checkout

Checkout gets the Branch from Github onto the Worker.
As the Worker is created everytime from Scratch every Job needs to get the Sources again.

## terraform cli

`   - uses: hashicorp/setup-terraform@v1`
Defines the Buildin Commands to be available on the worker from GitHub.

The CLI documentation can be found here.
https://learn.hashicorp.com/tutorials/terraform/azure-build?in=terraform/azure-get-started

## terraform init

Terraform Init sets up the Current Project Environment and connects to the Azure Storage Account Defined in your "`stage_1_AKS_Cluster/main.tf`"

## Terraform fmt
Terraform fmt allows you to Format your Code automatically so it matches the expected Syntax. It Beautyfies your code as well for better read ability. 

We set "`continue-on-error: true`" so you get Automatic Linting and the Pipeline doesn't allow properly Formatted code.

## Terraform Plan
Creates a Plan of the Changes needed to be done on Azure to accomplish the defined Settings in your Terraform Code.

## Dependencies and Environments

You can Setup Concurrent or sequential Tasks with Pipelines. In our example we made a Sequential Task by defining defining that the Seconds Task needs the First:

"`needs: [terraform]`"

Also we Setup two different Environments "production" and "development" those are necessary for an Approval Workflow. More on that later.


## Terraform apply

The Final Step is to Apply the Terraform Code which will Setup the Defined Environment on Azure.

## Terraform main.tf

Your /terrform/main.tf contains all the Setting for the Desired Infrastructure on Azure.

Every Terraform Project needs a Backend to Store the State by default a Local file will be used but there are many Different Available Backends. In our Case we Provide you with an Azure Storageaccount.
Which is Defined by first Stating the Resource Group and Storage Account Name. Inside a Storage Account we also Specify an Existing Container Name. The here Selected Key is the State File Terraform will use. If it doesnt exist it will be created by the Terraform CLI. 
The Name of the Key needs to be Unique for every Participant. (only Lowercase and numbers allowed)

Also we define the Version of the AzureRM Provider

```
terraform {
    required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=2.46.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "meta"
    storage_account_name = "hackathonterraform"
    container_name       = "tfstate"
    key                  = "########.tfstate"
  }
}
```

### Storage Account

To learn more about Azure Storage Accounts checkout:

https://docs.microsoft.com/de-de/azure/storage/common/storage-account-overview
### Backends

To learn more about Terraform Backends checkout:
https://www.terraform.io/docs/language/settings/backends/index.html

### State

To learn more about the Terraform State Checkout:
https://www.terraform.io/docs/language/state/index.html

## Data and Resource

There are different types of definitions we use to define Resources in the Code.

### Locals

Locals are local Variables
### Data

Used to get available Resources on Azure and read out there current Configuration for later Use. Data is never Altered by Terraform as it is an External Resource.

### Resource

A Resource is a Managed object by Terraform if terraform finds a difference in the Terraform State or the Actual Status of the Resource in Azure it will create a Plan on how to alter the State so it matches the Definitions in your Terraform Code again. 

There are Mandatory and Optional Settings for each resource Type.
## Resource Group

We define the following resource Group as a Resource for later use. In our Example everyone will need a different Resource Group Name.
Alter your Resource Group Name in the Locals.

locals {
  name     = $env.
  location = "West Europe"
}

```

resource "azurerm_resource_group" "global" {
  name     = var.uniquename
  location = local.location
}

```

More about Azure Resource Groups:
https://docs.microsoft.com/de-de/azure/azure-resource-manager/management/manage-resource-groups-portal

More About Terraform Resource Group Definitions:

https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group

## Defining a AKS

An AKS (Azure Kuberentes Cluster) is a Managed Kuberenetes Cluster in Azure. 

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

# 2. Run your Pipeline

After you Set up your Secrets and fixed the Code in your Repository.
You can try to run your Workflow.
To do so go to Actions and select the Terraform workflow on the Left site.

Now Select Run workflow on the Right side.

<br><img src="./images/runWorkflow.PNG" width="800"/><br>

## Workflow Progress

Wait for your Workflow to finish.
If the Task does not run through you may ask one of us to Help you out.
