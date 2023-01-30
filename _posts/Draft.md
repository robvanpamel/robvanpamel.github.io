Last week me and my colleagues went to NDC London. NDC conference is a 3 day conference where you can follow talks related to several topics like architecture, cloud, testing, .NET and many more
Each day you can choose between 7 different tracks, whatever triggers you the most. There are workshops available during the conference, if you like hands-on.  
My goal was to get more up to date with latest architecture patterns and look at some AWS tracks. 

My 3 favourite sessions were the ones listed below:

# Intentional Code - Minimalism in a World of Dogmatic Design by David Whitney
During this talk David told us more how we could better design our applications. This time not in the sense of design patterns or best practices but more to the fundamentals. Look at how we as a developer react when we see a code file. Do we want to close it, or can we understand the flow at a glance? 
Most of the examples aren't hard to grasp or complicated, but so easy to overlook them. It starts very easy for example by adding some newlines to the file, or I  call it, "Give your code some air to breathe". This can already improve the readabiliity a lot! 
But you can go a few steps forward, can this code be minified and simplified ? Look closer if you really need that additional level of abstraction which was added. Most likely you can remove it and don't need it. This lowers the cognitive load, makes it easier to changes which leads in the end again to a better architecture.   

# A perfect match: Dapr & Azure Container Apps by Sander Molenkamp
In this talk I got a good overview of the different possibilities that Dapr provides and how this can be used in combination with Azure Container Apps. My knowledge of Dapr was very limited so it was easy to overwhelm me. 
You can use it for services like PubSub, State Management, Secret Management, .... You can use the Dapr default, but in combination with Azure, you can hook it up with Azure Services like Azure Service Bus, Blob storage etc. Which makes it very powerfull in my opinion. A service that AWS is missing I think. 

# Don’t Build a Distributed Monolith: How to Avoid Doing Microservices Completely Wrong by Jonathan “J.” Tower
Jonathan provided the audience with a top 10 of things to avoid when building microservices. Although I think almost all of them can be found in the book of "Building Microservices" by Sam Newman, I still think this a good session! 
Let me sum some of them up for you. To learn all of them you should go and listen to his talk 
- Assuming Microservices are always better 
- Shared database between microservices
- Microservices are too small 
- Starting your microservices from scratch
- Coupling through cross cutting concerns
- Use of sync communications 
