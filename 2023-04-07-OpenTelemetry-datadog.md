---
title: Enable Datadog unified tagging using opentelemetry in Azure
date: 2023-03-07 
excerpt_separator: <!--more-->
tags: OpenTelemetry Datadog dotnet Azure
---
<p align="center">
  <img src="https://raw.githubusercontent.com/robvanpamel/robvanpamel.github.io/main/_posts/otel/observability.jpg" width="500">
</p>
Lately I was improving the observability of a project where I am working on. Like I described in one of the previous posts, this project is using OpenTelemetry to send the data to Datadog. 

Datadog has a feature which is called Unified Service Tagging which is explained HERE (https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes). This directly attaches your traces to the infrastructur, which is really cool! 

When you are using open telemetry you don't use datadogs native integration library so this feature is not enabled out of the box. 
By setting 3 open telemetry tags you can enable this feature and you do have the coupling between infrastructure and your APM traces! 
I've also tried to add service, version, ...tags to the infrastructure so that datadog would be able to 'magically' resolve this, but without luck. 

Okay that was it for today, a smaller post but I do think it was worth mentioning! 