---
title: Managed Identities in Azure
date: 2022-10-26   
excerpt_separator: <!--more-->
tags: Azure Managed Identities
---

When you want to access a resource in Azure like a storage account or a SQL database, there are multiple. SharedAccessKeys and connection strings are the most popular options we have nowadays. Who hasn't used a connectionstring to his sql-database?  
However these solutions are based on what you could call "shared credentials" which are not always the most secure way. You have to store the credentials in safe manner, so it doesn't get compromised.  
 
This is where Azure Key Vault is the obvious solution. The credentials which must remain secret can be stored in a secure way in the key vault. However it is already stored safely, there is still a possibility that it leaks or gets compromised by human errors. 
On top of that, you still need to rotate the secrets which puts load on the responsible team.  

 <!--more--> 
However in most cases the resource which we want to access runs in the same Azure environment as the resource that would like to access it. So it should be possible that we provide access without the hassle of shared credentials etc. 
The answer on this challenge is to use Managed Identities. 

## Azure Managed Identities
Managed identities make it possible to connect multiple resources without the management of secrets or credentials. you don't even have access to the secret or credential which is used. 
Azure mananaged identities can be used for each service or resource that supports [Azure AD authentication](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/managed-identities-status). And best past for last, the use of them is for free! 

### User-assigned Managed Identity
A user-assigned managed identity is an identity which can be shared accross multiple resources. It is not attached to a given resource, which means that multiple resources, can use the same Managed Identity to access a given resource. 

### System-assigned Managed Identity
A system-assigned managed identity is an identity which is directly attached to a given resource. It is created together with the resource and if the resource is removed, the managed identity is removed as well. 

# References
- https://learn.microsoft.com/en-us/azure/active-directory/authentication/overview-authentication
- https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
- https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/managed-identity-best-practice-recommendations
