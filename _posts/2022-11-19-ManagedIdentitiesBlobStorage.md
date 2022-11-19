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
Let's start with the Azure Function. The Azure function requires at least an App Service Plan and a 