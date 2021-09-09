# Setup Deployment Pipeline Environment Variables

To Connect your GitHub Pipeline to Azure you will need to Provide the required Credentials. As it is a Bad idea to Store it in Plain text in the Code you should always use a seperate Key Store.
In our Example we Store the Credentials as a Repository Secret.

## How to create a Secret

https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository

## Example Code Secrets

In our Provided Example Code we use the Following Secrets to configure our Project

To avoid name Collisions when creating our Resources on Azure in the Following Tasks, we need to create Unique Names for everyone.
Sometimes Naming Conventions  
### Prefix:

Best Practice Prefixes for Specific Azure Resources:

https://docs.microsoft.com/de-de/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations

### Suffix:

We all are in the same Subscription and use same Resource Group but everybody uses his own App Service Plan and his own Webapp.

Therefore, pick unique names for your App Service Plan and Webapp variables.

You may attach your own Suffix like a short form of your name and a random Number.
If your name is Florian Peters for Example you could choose:

`flopet631`

The Resource Name would then be `plan-flopet631`

In Real Customer Environments there are usually more detailed guidelines on how to name Resources.

### AZURE_CREDENTIALS

This is the Connection Data needed for the Azure Subscription.

For this Hackathon we will provide you with it:

Secret:
AZURE_CREDENTIALS =
`{
  "clientId": "#",
  "clientSecret": "#",
  "subscriptionId": "#",
  "tenantId": "#"
}`

### RG - ResourceGroup

This is the Name of the Resource Group you will be using to Deploy your Website. During the Hackathon you will only have Access to the Following ResourceGroup:

Secret:
rg = 
`ws-devops`

If you want to know more about Resource Groups take a look here:
https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal#what-is-a-resource-group


### ASP - AppServicePlan

This is the Name of the App Service Plan which needs to be unique in our Subscription.

Secret:
asp=
`plan-[your suffix]`

If you want to know more about App Service Plans take a look here:
https://docs.microsoft.com/en-us/azure/app-service/overview

### WEBAPP - WebApplication

This is the Name of the Web Application which needs to be unique globally.

Secret:
webapp=
`your web app name`

If you want to know more about WebApps take a look here:
https://docs.microsoft.com/en-us/azure/app-service/overview

# Good Luck and have Fun!
