# Setup Deployment Pipeline Environment Variables
To connect your GitHub pipeline to Azure you will need to provide the required credentials. As it is a bad idea to store it in plain text in the code you should always use a seperate key store. In our example we store the credentials as a repository secret.

## How to Create a Secret
https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository

## Example Code Secrets
In our provided example code we use the following secrets to configure our project.

## Naming Convention
To avoid name collisions when creating our resources on Azure in the following tasks we need to create unique names for everyone. That is why we are using naming conventions.

We all are in the same Azure Subscription and use same Resource Group but everybody uses his own App Service Plan and his own Webapp. This can lead to confusion. Therefore, pick unique names for your App Service Plan and Webapp variables.

Choose your prefixes and suffixes wisely, e.g. by following the [Best practice prefixes for specific Azure resources](https://docs.microsoft.com/de-de/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations).

You may attach your own suffix like a short form of your name and a random number. Assuming your name is Florian Peters you could choose: `flopet631`. The resource name would then be `plan-flopet631`

In real customer environments we usually use more detailed guidelines on how to name resources but this example will be sufficient for the hackathon.

### AZURE_CREDENTIALS
This is the connection data needed for the Azure Subscription.

For this Hackathon we will provide you with it. Here is the template:

**Secret:** *AZURE_CREDENTIALS=*
```
{
  "clientId": "#",
  "clientSecret": "#",
  "subscriptionId": "#",
  "tenantId": "#"
}
```

### AZURE_AD_CLIENT_ID
This is the connection data needed for the Azure Subscription.
**Secret:** *AZURE_AD_CLIENT_ID=`#`* 

### AZURE_AD_CLIENT_SECRET
This is the connection data needed for the Azure Subscription.
**Secret:** *AZURE_AD_CLIENT_SECRET=`#`* 

### AZURE_AD_TENANT_ID
This is the connection data needed for the Azure Subscription.
**Secret:** *AZURE_AD_TENANT_ID=`#`* 

### AZURE_SUBSCRIPTION_ID
This is the connection data needed for the Azure Subscription.
**Secret:** *AZURE_SUBSCRIPTION_ID=`#`*

### UNIQUE_NAME
This is the name for all of your resources on Azure. Please provide a unique team name. It should consist of a maximum of 7 lower case letters without numbers of special characters. 

**Examples:**
- Team one: *first*
- *Secret: *UNIQUE_NAME=`#`*

# Result
<br><img src="./images/SecretOverview.PNG" width="400"/><br>

# Whats Next?
Continue with the next step on the [Main Page](README.md)