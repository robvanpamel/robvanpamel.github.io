---
title: Managed Identities in Azure with blob storage
date: 2022-11-20 
excerpt_separator: <!--more-->
tags: Azure Managed Identities
---
In my previous blog post, you could already see which benefits Managed Identities have. As mentioned over there, they increase the security inside your Azure environment. Now we will take this theorie into practice and start working with it. We'll create an azure function which access a storage account by using the user Managed Identity.
 <!--more-->

There will be 2 parts in this blogpost, the first one is to setup up the Azure environment, creating the azure function, the managed Identities etc. The second part where the application will be updated to use the User Managed Identity.

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

Once the function is created, the next step is to create a User Managed Identity, a Role and a RoleAssignment. 

The Role is created below. We are using an existing role. 

    @description('This is the built-in Storage Blob Data Contributor role. See https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#contributor')
    resource contributorRoleDefinition 'Microsoft.Authorization/roleDefinitions@2018-01-01-preview' existing = {
      scope: subscription()
      name: 'ba92f5b4-2d11-453d-a403-e96b0029c9fe'
    }