---
title: Managed Identities in Azure
date: 2022-10-26   
excerpt_separator: <!--more-->
tags: Azure Managed Identities
---

When you want to access a resource in Azure like a storage account or a SQL database, there are multiple. SharedAccessKeys and connection strings are the most popular options we have nowadays. Who hasn't used a connectionstring to his sql-database?
However, these solutions are based on what you could call "shared credentials" which are not always the most secure way. You have to store the credentials in safe manner, so it doesn't get compromised.

This is where Azure Key Vault is the obvious solution. The credentials which must remain secret can be stored in a secure way in the key vault. However, it is already stored safely, there is still a possibility that it leaks or gets compromised by human errors. On top of that, you still need to rotate the secrets, which puts load on the responsible team.

However, in most cases, the resource which we want to access runs in the same Azure environment as the resource <!--more--> that would like to access it. So it should be possible that we provide access without the hassle of shared credentials etc. The answer to this challenge is using Managed Identities.  

# Azure Managed Identities
Managed identities make it possible to connect multiple resources without the management of secrets or credentials. you don't even have access to the secret or credential which is used. 
Azure mananaged identities can be used for each service or resource that supports [Azure AD authentication](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/managed-identities-status). And best past for last, the use of them is for free! 

### User-assigned Managed Identity
A user-assigned managed identity is an identity which can be shared accross multiple resources. It is not attached to a given resource, which means that multiple resources, can use the same Managed Identity to access a given resource. 

### System-assigned Managed Identity
A system-assigned managed identity is an identity which is directly attached to a given resource. It is created together with the resource and if the resource is removed, the managed identity is removed as well. 

Managed identities, system or user-assigned, will only define who can do an action. What and where it can be done, is not defined yet. This is defined inside a Role Assignment. 

## Role assignments
A role assignments in azure is a definition of 3 principles
- What can be done - is defined in the role 
- Who can do it - is defined in the principal 
- Where can it be done - is defined in the context.

### Role 
Like mentioned above, the role will define what can be done in a list of permissions, eg the storage blob data reader can access the blob storage for read, but he is not able to write to it. Multiple built-in roles are available, but custom roles can be created as well.

### Principal
The principal can be a managed identity, which we discuss currently, but it can also be a user or a group. 

### Scope 
The scope is where the user can use the role, For example in the given resource group, or in the subscription.

Role assignments are maintained individually as well as for user as for system-assigned identities. This implies that in case that a system or a user assigned identity is removed, the role assignment still needs to be  updated. 

## Deployments and Managed Identities
Although both managed identities can serve the same needs, user-assigned Managed identities have my preference. This is mainly for deploying resources and the Managed Identities for that resource. When using system-assigned managed identities, the number of role assignments which are required can increase very rapidly. Be aware that there is a limit when creating these assignments. 

![system-assigned identities*](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/media/managed-identity-best-practice-recommendations/system-assigned-identities.png)*system assigned identities - 8 role assignements*

In the example above (image from microsoft) already 8 role assignments need to be created. When we are working with User Based identities only 2 are required, see below. This already makes the life a bit easier. 

![user-assigned identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/media/managed-identity-best-practice-recommendations/user-assigned-identities.png)*user assigned identities - 2 role assignements*

Combining multiple user-assigned Managed Identities is a flexibility that is only possible when working with user-assigned managed identities. We could combine a ‘shared user-assigned identity’ with a more specific user-assigned identity. Although the latter is also possible when combining system and user-managed identities.

It is important to note that combining several user-assigned identities should still adhere to the principle of the least privilege.

## Use of user-assigned or system-assigned managed identities?
The preference here goes to the usage of user-assigned above system-assigned managed identities. As they are more flexible to be used across multiple resources it will improve the developer experience, while it isn’t less secure compared to system-assigned managed identities. 

User-assigned managed identities have their own lifecycle, what improves creation of resources and resource groups. System-assigned managed identities could require multiple runs of a pipeline before a role-assignment can be done, because you can't create a role assignment for a resource which isn’t created yet.

That's it for today, thanks for reading along! 
In the next blog post I 'll explain more in detail how you can create a user-assigned identities and how to use them to access a storage account from an appservice. See you there!

# References
- https://learn.microsoft.com/en-us/azure/active-directory/authentication/overview-authentication
- https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
- https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/managed-identity-best-practice-recommendations
