# Infrastructure Repository and Pipeline

## Introduction

You should now have Completed the Following things:
1. Setup your GitHub Account
2. Setup the Example Code in your Account
3. Added the Repository Secrets to the Example Code

Next you will get an Overview over the Example Code to Setup your first own Website on Azure via Terraform. This will only be the most basic Infrastructure Setup. In a later Step we will then alter the Websites Content.

If you want to learn more about the concept of a pipeline you can do it here:

[https://docs.github.com/en/actions/quickstart](https://docs.github.com/en/actions/quickstart)


# 1. Setting up the Infrastructure Pipeline

The first Step in Creating a Pipeline with GitHub Actions would normally be to Select a Template and Start from there.

To do so you would go to Actions in your Code Repository and select an appropriate Template or start from Scratch with an empty one.

In our Example we already Created the necessary Files. So to say our own Template.

Your first Task is to go into your Repository and look at the Following file.

>_Warning: The formatting of YAML (yml) files is based on spaces and tabs and therefore the following lines should be copied with care.
> It is advised to use Visual Studio Code to validate the copied file._
> <br> [How to work with Git Locally](/01.5_SetupGit.md)


`#File: .github/workflows/azure_tf.yml`
```
name: 'Terraform'
 
on:
  workflow_dispatch

env:
  ARM_CLIENT_ID: ${{ secrets.AZURE_AD_CLIENT_ID }}
  ARM_CLIENT_SECRET: ${{ secrets.AZURE_AD_CLIENT_SECRET }}
  ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  ARM_TENANT_ID: ${{ secrets.AZURE_AD_TENANT_ID }}

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest
    environment: developement
 
    # Use the Bash shell regardless whether the GitHub Actions runner is ubuntu-latest, macos-latest, or windows-latest
    defaults:
      run:
        shell: bash
        working-directory: terraform
 
```

## Azure Cli

The Pipeline we are Proposing here is using Terraform to create the Service Plan on Azure.

Github Terraform Doc: 
<br> https://github.com/hashicorp/setup-terraform

Azure AppServicePlan and WebApp: 
<br> https://docs.microsoft.com/en-us/azure/app-service/overview

## Pipeline Name

Next we Specify the Name of our GitHub Action in our Example "`name: infra`".

## Triggers

The Code Starts by using "`on: workflow_dispatch`" which means one of the Triggers to Start this Pipeline is to do it Manually.

There are many Automatic triggers you can use, to learn more about Triggers check this:
https://docs.github.com/en/actions/reference/events-that-trigger-workflows#workflow_dispatch

## Environment Variables:

The Environment Variables are set in the Code Runners the Pipeline uses. In our Example these are known Variables by Terraform which it uses to Authenticate against Azure.
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

Terraform Init sets up the Current Project Environment and connects to the Azure Storage Account Defined in your "`terraform/main.tf`"

## Terraform fmt
Terraform fmt allows you to Format your Code automatically so it matches the expected Syntax. It Beautyfies your code aswell for better readablity. 

We set "`continue-on-error: false`" so you get Automatic Linting and the Pipeline doesnt allow properly Formatted code.

## Terraform Plan
Creates a Plan of the Changes needed to be done on Azure to accomplish the defined Settings in your Terraform Code.

## Dependencies and Environments

You can Setup Concurrent or sequential Tasks with Pipelines. In our example we made a Sequential Task by definining defining that the Seconds Task needs the First:

"`needs: [terraform]`"

Also we Setup two different Environments "production" and "developement" those are necessary for an Approval Workflow. More on that later.


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

There are Mandetory and Optional Settings for each resource Type.
## Resource Group

We define the following resource Group as a Resource for later use. In our Example everyone will need a different Resource Group Name.
Alter your Resource Group Name in the Locals.

locals {
  name     = "yourname"
  location = "West Europe"
}

```

resource "azurerm_resource_group" "global" {
  name     = local.name
  location = local.location
}

```

More about Azure Resource Groups:
https://docs.microsoft.com/de-de/azure/azure-resource-manager/management/manage-resource-groups-portal

More About Terraform Resource Group Definitions:

https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group


# 2. Run your Pipeline

After you Set up your Secrets and fixed the Code in your Repository.
You can try to run your Workflow.
To do so go to Actions and select the Terraform workflow on the Left site.

Now Select Run workflow on the Right side.

<br><img src="./images/runWorkflow.PNG" width="800"/><br>

## Workflow Progress

Wait for your Workflow to finish.
If the Task does not run through you may ask one of us to Help you out.
## Check your WebApp is online after approx. 5 minutes

https://`[yourWebAppName]`.azurewebsites.net/

You should see a &quot;Hey, Node developers&quot; welcome screen.

Congratulations, you have deployed your first WebApp infrastructure.
 Now, you can go ahead and deploy some code to your WebApp.