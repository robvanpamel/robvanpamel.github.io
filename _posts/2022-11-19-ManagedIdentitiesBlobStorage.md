---
title: Managed Identities in Azure with blob storage
date: 2022-11-20 
excerpt_separator: <!--more-->
tags: Azure Managed Identities
---
In my previous blog post, you could already see which benefits Managed Identities have. As mentioned over there, they increase the security inside your Azure environment. Now we will take this theorie into practice and start working with it. We'll create an azure function which access a storage account and writes a stream to it, by using the user Managed Identity.
 <!--more-->

There will be 2 parts in this blogpost, the first one is to setup up the Azure environment, creating the azure function, the managed Identities etc. The second part where the application will be updated to use the User Managed Identity to write the stream to the blobl storage.

## The Azure environment 

Creating the Azure environment will be done in combination with Bicep. 
Let's start with the Azure Function. The Azure function requires at least an App Service Plan, a storage account and off course a function app. 

The service plan is create on the `Dynamic` tier and as `reserved` which enabled running on a a Linux environment. 

    resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
        location: location
        name: 'asp-blog-managed-identities'
        kind: 'linux'
        sku:{
            name: 'Y1'
            tier: 'Dynamic'
        }
        properties:{
            reserved: true
        }
        tags:{
            blog: 'blog-managed-identities-storage'
        }
    }

The function app and its linked storage account is described below. The functionapp is attached to the App Service Plan via the `serviceFarmId`. The AzureWebJobsStorage which is listed in the appSettings, only purposes the storage where the binaries of the function is stored. It isn't involved in the futher process of accessing data via a user managed identity. 

    resource functionApp 'Microsoft.Web/sites@2022-03-01'={
    name: 'fct-blobwriter'
    location: location
    kind: 'functionapp'
    identity: {
        type: 'UserAssigned'
        userAssignedIdentities: {
            '${usermanagedIdentity.id}': {}
        }
    }  
    properties:{
        serverFarmId: appServicePlan.id
        siteConfig: {
            appSettings: [
                {
                    name: 'AzureWebJobsStorage'
                    value: 'DefaultEndpointsProtocol=https;AccountName=${storageAccountName};EndpointSuffix=${environment().suffixes.storage};AccountKey=${storageAccount.listKeys().keys[0].value}'
                }
                {
                    name: 'FUNCTIONS_EXTENSION_VERSION'
                    value: '~4'
                }
                {
                    name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
                    value: applicationInsights.properties.InstrumentationKey
                }
                {
                    name: 'FUNCTIONS_WORKER_RUNTIME'
                    value: 'dotnet-isolated'
                }
                {
                    name: 'AZURE_CLIENT_ID'
                    value: usermanagedIdentity.properties.clientId 
                }    
            ]
            minTlsVersion: '1.2'
        }
        httpsOnly: true
    }
    }

    resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
        name: storageAccountName
        location: location
        sku: {
            name: storageAccountType
        }
        kind: 'StorageV2'
        properties: {
        }
        tags:{
            blog: 'blog-managed-identities-storage'
        }
    }

In the identity section, an array of user managed identities are added. In this example only one is added. 

    identity: {
        type: 'UserAssigned'
        userAssignedIdentities: {
            '${usermanagedIdentity.id}': {}
        }
    }  

Next to this, which user managed identity to use, is specified in the appsettings of the function.

    {
        name: 'AZURE_CLIENT_ID'
        value: usermanagedIdentity.properties.clientId 
    }


Once the function is created, the next step is to create a User Managed Identity, a Role and a RoleAssignment. 

The Role is created below, for simplicity an existing is being used here. You can find a full list with their corresponding guid over here https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#contributor 


    @description('This is the built-in Storage Blob Data Contributor role. See https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#contributor')
    resource contributorRoleDefinition 'Microsoft.Authorization/roleDefinitions@2018-01-01-preview' existing = {
      scope: subscription()
      name: 'ba92f5b4-2d11-453d-a403-e96b0029c9fe'
    }

Once these are created, it is time to add the RoleAssignment. In this Role Assignment the User Managed Identity and the Role definition are coupled together.  

    resource roleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
        name: guid(resourceGroup().id, usermanagedIdentity.id, contributorRoleDefinition.id)
        properties:{ 
            principalId: usermanagedIdentity.properties.principalId 
            roleDefinitionId: contributorRoleDefinition.id
        }
    }


Now it is time to move over to the application to start using this. 

## The Application 

Before starting using the User Managed Identity, let see how this should typical is done without them. 

### Without User Managed Identity
When no Managed Identity is used, a SAS token is created on the Azure portal to gain access to the blobcontainer. An important aspect to get the SAS token from the Azure Portal, is to store it in a safe manner, eg Azure Keyvault.

    public static async Task WriteWithSas()
    {
        var connectionString =
                $"BlobEndpoint=https://{BLOB_ACCOUNT}.blob.core.windows.net/;{BLOB_SASTOKEN}";
        var client = new BlobContainerClient(connectionString, BLOB_CONTAINER);
        ... 
        ... // stream stuff goes here
        ...
        return  await blobContainerClient.UploadBlobAsync($"file-{DateTime.UtcNow:yyyy-MM-dd-HH-mm-ss}.txt", stream);
    }

### Using User Managed Identity
Now when using the User Managed Identity, we don't have to securely fetch any identities or so, we can just safely use it, which is the whole idea to make it much safer. 
Now you'll notice that there is no SAS token, or another secret involved when creating the connection string. The difference between those, is to use the DefaultAzureCredential. This defaultAzureCrendential will use the user managed identity which is specified in the appsettings of the function `AZURE_CLIENT_ID`. 

        string containerEndpoint = $"https://{BLOB_ACCOUNT}.blob.core.windows.net/{BLOB_CONTAINER}";

        var client = new BlobContainerClient(new Uri(containerEndpoint), new DefaultAzureCredential());
        ... 
        ... // stream stuff goes here
        ...
        return  await blobContainerClient.UploadBlobAsync($"file-{DateTime.UtcNow:yyyy-MM-dd-HH-mm-ss}.txt", stream);
        
Using this the Azure function is allowed to access and write a stream to the blob storage! 

The complete example can be found over here https://github.com/robvanpamel/blogs-code/tree/main/2022-ManagedIdentities

If you have any questions or comments please leave them over [here](https://github.com/robvanpamel/robvanpamel.github.io/issues/new) 

Thanks for reading! 