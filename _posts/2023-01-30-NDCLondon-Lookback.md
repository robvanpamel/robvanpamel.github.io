---
title: Looking back to NDC London 2023
date: 2023-01-30 
excerpt_separator: <!--more-->
tags: NDC Education learning 
---

Last week my colleagues and I went to the NDC conference in London. This NDC conference is a 3 day conference where you can attend several talks related to different topics like architecture, cloud, testing, .NET and many more.

Each day you can choose between 7 different tracks, whatever triggers you the most. If you like hands-on, there are workshops available during the conference.   
My goal was to get more up to date with latest architecture patterns and look at some AWS tracks. 

3 of my favourite sessions are listed below:<!--more--> there is much more good content, but I can't list them all off course. 

### Intentional Code - Minimalism in a World of Dogmatic Design by David Whitney
During this talk David told us how we could create a better design for our applications. Not design in the sense that you should add design patterns or best practices but more to the fundamentals. Looking at how we as a developer feel when we look at code. Do we want to close the file, or can we understand the flow at a glance? 
Most of the examples aren't hard to grasp or complicated, but it is so easy to overlook them. It starts very easy for example by adding some newlines to the file, or I  call it, "Give your code some air to breathe". This can already improve the readabiliity a lot! 
But you can go a few steps forward, can this code be minified and simplified? Look closer if you really need that additional level of abstraction which was added. Most likely you can remove it and don't need it. This lowers the cognitive load, makes it easier to changes which leads in the end again to a better architecture. The talk continues with more examples and a good reading reference would be "Code that fits in your head".

### A perfect match: Dapr & Azure Container Apps by Sander Molenkamp
In this talk I got a good overview of the different possibilities that Dapr provides and how this can be used in combination with Azure Container Apps. My knowledge of Dapr was very limited so it was easy to overwhelm me. 
You can use Dapr for enabling services like PubSub, State Management, Secret Management, .... You can use the Dapr defaults, but in combination with Azure, you can hook it up with Azure Services like Azure Service Bus, Blob storage etc. Which makes it very powerfull in my opinion. A service that AWS is missing I think, although you can add Dapr yourself. 

### Don’t Build a Distributed Monolith: How to Avoid Doing Microservices Completely Wrong by Jonathan “J.” Tower
Jonathan provided the audience with a top 10 of things to avoid when building microservices. Although I think almost all of them can be found in the book of "Building Microservices" by Sam Newman, I still think this a good reminder! 
Let me sum some of them up for you. To learn all of them you should go and listen to his talk 
- Assuming Microservices are always better 
- Shared database between microservices
- Microservices are too small 
- Starting your microservices from scratch
- Coupling through cross cutting concerns
- Use of sync communications 


Once again, there were a lot of other interesting sessions but it would take way too long to list all these talks. I must admit that the overall quality of the given talks on the conference was very high. And looking back also includes the nice lunches we had in combination with a good party from the line-breakers on Thursday. 

That will be it for today, see you next time! 
