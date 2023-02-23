---
title: Find your application hidden secrets using opentelemetry
date: 2023-02-23 
excerpt_separator: <!--more-->
tags: OpenTelemetry Datadog dotnet 
---
<p align="center">
  <img src="https://raw.githubusercontent.com/robvanpamel/robvanpamel.github.io/main/_posts/otel/observability.jpg" width="500">
</p>
When your project is growing, the need to gain more insights in your application is growing along. You need to know in detail what is happening inside the application, either to resolve a bug or to improve some performance issues. In the past these insights were mainly received by adding extensive logging. The most of us have already done this, and is the easier step to start with. 
If you would like to go a step further, adding trace and metric information is the way to go. Trace information is extremely valuable when you start to work with distributed applications. They allow you to follow a request across multiple systems. <!--more-->

The combination of logs, traces and metrics are called telemetry data. 
While there are different ways to collect the data above, OpenTelemetry should be the defacto-standard these days. Opentelemetry is a Cloud Native Computing Foundation (CNCF) project because it provides us with an open standard to collect telemetry data from our applications. _(the standard isn't fully approved yet, but this shouldn't hold us from using it)_

The open telemetry projects consists out of several topics which i would like to explain
- Signals
- Open Telemetry Collector
- Instrumenting your application

# Signals

In open telemetry, the signals are the actual collection of different telemetry data that are supported, logs, traces and metrics. 

## Logs 
A log is a timestamped text record, either structured (recommended) or unstructured, with metadata. 

## Traces 
Traces give us the big picture of what happens when a request is made by a user or an application. For example a GET request was executed.

## Metrics 
A metric is a measurement about a service, captured at runtime.


# Open Telemetry Collector 

The open telemetry Collector is the process which will act as a gateway to receive the signals (telemetry data), process it and send to your observability tool. Using the open telemetry collector isn't a mandatory step. It is possible to send the telemetry data to your observability tool, however this isn't recommended for production environments. The collector can take care of retries, TODO  and more.  
There are 2 collector variants available, the _'[normal](https://github.com/open-telemetry/opentelemetry-collector/releases)'_ and the _[contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib/releases)_. As you expected, the contrib variant includes more receivers, exporters and processors built by the community.   
<p align="center">
  <img src="https://raw.githubusercontent.com/robvanpamel/robvanpamel.github.io/main/_posts/otel/otel_collector.svg" width="400">
</p>

## Receivers
Here you can define how you will ingest data into your collector, it can be one source, but nothings holds you from using multiple receivers. The default is the Open Telemetry Protocol (OTLP). OTLP runs on HTTP(4318) and on gRPC (port 4317). Other options which are out-of-the-box available are Jaeger and Prometheus, but if you use the contrib variant, you have even more options.  
- https://opentelemetry.io/docs/collector/configuration/#receivers 
## Processors
Here is where you can add some magic to your traces. Each trace, log or metric can be tweaked in this space. Some examples are adding tags, filtering logs or traces, sample, ... More information can be found here. 
- https://opentelemetry.io/docs/collector/configuration/#processors 

## Exporters
Like the name mentions, the exporter is responsible for exporter the telemetry data into your observability tool. Just like at the receiver side, the default is the Open Telemetry Protocol (HTTP and gRPC), but also Jaeger and Prometheus are present. And when using the contrib variant of the collector, much more is possible. In the example below we will be using the datadog exporter, cause we would like to import our data into it.  
- https://opentelemetry.io/docs/collector/configuration/#exporters

## Configuration 

The configuration of the Collector is done via a `yaml` file, in which you can define how it works, lets take a look. 

First part that we need to configure is the receiver side. In our example, we will use the OTLP with http and gRPC enabled. We will have to instrument our web-application as well that we 'export' the OTLP data over there. For more information see [Instrumenting your application](#instrumenting-your-dotnet-core-application)

````
receivers:
  otlp:
    protocols:
      http:
      grpc:
```` 

The next part to configure is the processor side. Like mentioned above you can tweak your signals here before sending them to your exporter. The example above has 2 processors. 
One thing to note over here is that not all the processors are already in a stable phase. I personally don't mind it as, but you should be aware of this. See [this link for more information about the stability levels ](https://github.com/open-telemetry/opentelemetry-collector#stability-levels) 

The first processor is a filter that we defined for our traces. In the example below we have defined some healthchecks on our application, and we do not want them to 'pollute' our traces when everything is configured as it should be (HTTP status = 200). By adding this filter, we can remove these traces, but if the health check fails (HTTP status <> 500), we will still see it. This filter processor has lots of possibilities and examples which can be found [on the readme](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/filterprocessor). 


The next processor which is available is the batch processor. You can already expect what this does, it is batching the data into payloads before it is exported ( in this case to Datadog). Batches can be created based on size or on time.  
There is a time out (eg. 10s), after which a batch will be sent regardless of size. On the other hand, you have the batch size. This specifies the amount of signals a bath contains before being sent, send_batch_size. 

```` 
processors:
  filter: 
    traces: 
      span: 
       - 'attributes["http.target"] == "/health/ready" and attributes["http.status_code"] == 200'
       - 'attributes["http.target"] == "/health/live" and attributes["http.status_code"] == 200'
  
  # The batch processor batches telemetry data into larger payloads.
  # It is necessary for the Datadog traces exporter to work optimally,
  # and is recommended for any production pipeline.
  batch:
    # Datadog APM Intake limit is 3.2MB. Let's make sure the batches do not
    # go over that.
    send_batch_max_size: 1000
    send_batch_size: 100
    timeout: 10s
```` 

The next part in the configuration, are the exporters. Here you specify where your data will be sent to. I have specified 2 exporters here, a file exporter and also our datadog exporter. [Here](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter) you can find a list of all the exporters provided by the community.
For each exporter you have to specify a bit more content, eg the file exporter requires a path to be defined, where the datadog exporter requires a api-key and site. Each exporter has a descent readme file on github where the required attributes are listed. Here you can fine the readme for [datadog](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/datadogexporter)
```` 
exporters:
  file/no_rotation:
    path: ./logging
  datadog:
    api:
      ## The Datadog API key to associate your Agent's data with your organization.
      key: "<Your API key goes here>"
      site: datadoghq.eu
      fail_on_invalid_key: true
```` 

The last part to define in the collector is the pipeline. This is the place where you are connecting the dots. For each signal being a trace, metric or log, you can define its receiver, processor and exporter. This is where the power of the collector becomes visible. You can start to export your telemetry to different exporters, you can receive logs, only from specials sources that you output to other exporters, really nice.

```` 
service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [datadog]
    traces:
      receivers: [otlp]
      processors: [batch, filter]
      exporters: [datadog]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [datadog, file/no_rotation]
````

When you spin up the collector you will see something like this _(I manually filtered the output a bit here)_

````
C:\Users\rob.vanpamel\work\OTELCOL> .\otelcol-contrib.exe --config .\configuration\config.yaml
2023-02-23T10:24:56.365+0100    info    service/telemetry.go:90 Setting up own telemetry...
2023-02-23T10:24:56.369+0100    info    service/telemetry.go:116        Serving Prometheus metrics      {"address": ":8888", "level": "Basic"}
2023-02-23T10:24:56.712+0100    info    provider/provider.go:41 Resolved source {"kind": "exporter", "data_type": "metrics", "name": "datadog", "provider": "system", "source": {"Kind":"host","Identifier":"BEACC-8MT3MN3"}}
2023-02-23T10:24:56.718+0100    info    clientutil/api.go:47    Validating API key.     {"kind": "exporter", "data_type": "metrics", "name": "datadog"}
2023-02-23T10:24:56.967+0100    info    clientutil/api.go:51    API key validation successful.  {"kind": "exporter", "data_type": "metrics", 
2023-02-23T10:24:57.334+0100    info    clientutil/api.go:51    API key validation successful.  {"kind": "exporter", "data_type": "logs", "name": "datadog"}
2023-02-23T10:24:57.350+0100    info    logs/sender.go:45       Logs sender initialized {"kind": "exporter", "data_type": "logs", "name": "datadog", "endpoint": "https://http-intake.logs.datadoghq.eu"}
2023-02-23T10:24:57.357+0100    info    service/service.go:128  Starting otelcol-contrib...     {"Version": "0.70.0", "NumCPU": 16}
2023-02-23T10:24:57.357+0100    info    extensions/extensions.go:41     Starting extensions...
2023-02-23T10:24:57.358+0100    info    service/pipelines.go:86 Starting exporters...
2023-02-23T10:24:57.359+0100    info    service/pipelines.go:90 Exporter is starting... {"kind": "exporter", "data_type": "metrics", "name": "datadog"}
2023-02-23T10:24:57.360+0100    info    service/pipelines.go:94 Exporter started.       {"kind": "exporter", "data_type": "metrics", "name": "datadog"}
2023-02-23T10:24:57.360+0100    info    service/pipelines.go:90 Exporter is starting... {"kind": "exporter", "data_type": "traces", "name": "datadog"}"datadog"}
2023-02-23T10:24:57.362+0100    info    service/pipelines.go:98 Starting processors...
2023-02-23T10:24:57.362+0100    info    service/pipelines.go:102        Processor is starting...        {"kind": "processor", "name": "batch", "pipeline": "metrics"}AppInsights", "pipeline": "traces"}
2023-02-23T10:24:57.373+0100    info    service/pipelines.go:106        Processor started.      {"kind": "processor", "name": "filter/logs", "pipeline": "logs"}
2023-02-23T10:24:57.374+0100    info    service/pipelines.go:102        Processor is starting...        {"kind": "processor", "name": "batch", "pipeline": "logs"}
2023-02-23T10:24:57.374+0100    info    service/pipelin. 
...  
2023-02-23T10:24:57.383+0100    info    otlpreceiver@v0.70.0/otlp.go:112        Starting HTTP server    {"kind": "receiver", "name": "otlp", "pipeline": "logs", "endpoint": "0.0.0.0:4318"}
2023-02-23T10:24:57.385+0100    info    service/pipelines.go:118        Receiver started.       {"kind": "receiver", "name": "otlp", "pipeline": "logs"}
2023-02-23T10:24:57.385+0100    info    service/pipelines.go:114        Receiver is starting... {"kind": "receiver", "name": "otlp", "pipeline": "metrics"}
2023-02-23T10:24:57.387+0100    info    service/pipelines.go:118        Receiver started.       {"kind": "receiver", "name": "otlp", "pipeline": "traces"}
2023-02-23T10:24:57.388+0100    info    service/service.go:145  Everything is ready. Begin running and processing data.

````

# Instrumenting your dotnet core application 
When you want to collect traces, metrics and logs, you have to make some code changes to your codebase before they can be sent to eg Datadog with OpenTelemetry. Yes, there are possibilities to do the same without code changes, but you are not using the open telemetry standard at that moment, but a vendor specific implementation.  

## Collecting logs
Collecting logs is the easiest part, if you are already using the `ILogger` interfaces that dotnet core provides you.
The Nuget Packages to add to your project are listed below. Remember that they are still in beta/preview but it shouldn't hold you back.   
- [OpenTelemetry.Exporter.OpenTelemetryProtocol.Logs](https://www.nuget.org/packages/OpenTelemetry.Exporter.OpenTelemetryProtocol.Logs/1.4.0-rc.3)
- [OpenTelemetry.Exporter.Console](https://www.nuget.org/packages/OpenTelemetry.Exporter.Console/1.4.0-rc.3) 

After you've added the packages you need to extend the `startup.cs` file and extend the service collection. 

````
public void ConfigureServices(IServiceCollection services)
{
    ... // other service registrations etc
    services.AddLogging(loggingBuilder =>
        loggingBuilder.AddOpenTelemetry(otelLoggerOptions =>
        {
            otelLoggerOptions.IncludeFormattedMessage = true;
            otelLoggerOptions.IncludeScopes = true;
            otelLoggerOptions.ParseStateValues = true;
            otelLoggerOptions.AddConsoleExporter();
            otelLoggerOptions.AddOtlpExporter();
        })
    );
}
````

Afterwards you can log like you were doing already by injecting the `ILogger<>` or `ILoggerFactory`.
````
public class MyLoggingController
{
    public MyLoggingController(ILogger<MyLoggingClass> logger)
    {
        logger.LogInformation("The logging class is created");
    }
}
````


And the result should look like 
````
LogRecord.Timestamp:               2023-02-23T09:39:17.1029870Z
LogRecord.TraceId:                 87f9c6ac8d1cc7aea9c105c866a5095b
LogRecord.SpanId:                  a0d1d3ed1cbd01d0
LogRecord.TraceFlags:              Recorded
LogRecord.CategoryName:            MyApp.Controllers.v1.MyLoggingController
LogRecord.LogLevel:                Information
LogRecord.FormattedMessage:        The logging class is created
LogRecord.StateValues (Key:Value):
    OriginalFormat (a.k.a Body): The logging class is created
LogRecord.ScopeValues (Key:Value):
[Scope.0]:SpanId: a0d1d3ed1cbd01d0
[Scope.0]:TraceId: 87f9c6ac8d1cc7aea9c105c866a5095b
[Scope.0]:ParentId: 0000000000000000
[Scope.1]:ConnectionId: 0HMOLG7A3HTFA
[Scope.2]:RequestId: 0HMOLG7A3HTFA:00000001
[Scope.2]:RequestPath: /api/v1/default
[Scope.3]:ActionId: a94a60be-d09e-425e-85d9-c5111ca61cf5
[Scope.3]:ActionName: MyApp.Controllers.v1.MyLoggingController.GetAsync (MyApp)

Resource associated with LogRecord:
service.name: MyApp
service.instance.id: cbfe9734-9fe4-4711-9ea2-fdfe45f2251e

````


## Collecting Traces

Traces in your web-application are generated by default whenever your web-application receives a request. This means that you don't have to create them manually. You do have to catch them and send them to the Open Telemetry Collector. 

Enabling this for open telemetry can also be done by extending the services collection. You will see in the configuration that we enable 3 kind of instrumentations.  
Instrumentation for: 
- AspNetCore
- SqlClient 
- HTTPClient 

The required nuget packages are the ones below:

- [OpenTelemetry.Instrumentation.AspNetCore](https://www.nuget.org/packages/OpenTelemetry.Instrumentation.AspNetCore)
- [OpenTelemetry.Instrumentation.Http](https://www.nuget.org/packages/OpenTelemetry.Instrumentation.Http) 
- [OpenTelemetry.Instrumentation.SqlClient](https://www.nuget.org/packages/OpenTelemetry.Instrumentation.SqlClient) 
- [OpenTelemetry.Exporter.OpenTelemetryProtocol](https://www.nuget.org/packages/OpenTelemetry.Exporter.OpenTelemetryProtocol) 
- [OpenTelemetry.Exporter.Console](https://www.nuget.org/packages/OpenTelemetry.Exporter.Console) 
- [OpenTelemetry.Extensions.Hosting](https://www.nuget.org/packages/OpenTelemetry.Extensions.Hosting) 

Instrumentation will be added automatically afterwards. IF you think it is too much, you can start filtering in the collector. 

````
  var resourceBuilder = ResourceBuilder
                            .CreateDefault()
                            .AddService("MyService");
   
  services
    .AddOpenTelemetry()
    .WithTracing(openTelemetryBuilder =>
    {   
        openTelemetryBuilder
            .AddConsoleExporter()
            .AddOtlpExporter()
            .AddSource("MyService.*")
            .ConfigureResource(resourceBuilder)
            .AddAspNetCoreInstrumentation()
            .AddSqlClientInstrumentation(
                options => options.SetDbStatementForText = true
            )
            .AddHttpClientInstrumentation(
                options => options.RecordException = true);
                })
````

When you run the application with the above configuration, you notice that logs and traces are exported towards OTLP but also to your console. I think this console is still useful for local development.

The output on your console should look like this 

````
Activity.TraceId:            178afd54db82d620af89f346dd26eeae
Activity.SpanId:             1d3093c32ceb22be
Activity.TraceFlags:         Recorded
Activity.ActivitySourceName: OpenTelemetry.Instrumentation.AspNetCore
Activity.DisplayName:        api/v{version:apiVersion}/{tenant:length(1,100)}/blogs
Activity.Kind:               Server
Activity.StartTime:          2023-02-23T09:33:42.4927438Z
Activity.Duration:           00:00:07.9412154
Activity.Tags:
    net.host.name: localhost
    net.host.port: 44381
    http.method: GET
    http.scheme: https
    http.target: /api/v1/default/blogs
    http.url: https://localhost:44381/api/v1/default/blogs
    http.flavor: 2.0
    http.user_agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
    http.route: api/v{version:apiVersion}/{tenant:length(1,100)}/blogs
    http.status_code: 200
    Principal: 0aa00995-cb51-49ed-b2c6-5e5696170db9
Resource associated with Activity:
    service.name: MyApp
    service.instance.id: 43af6b39-2e41-475d-857c-3b3dd3581748
````


## Remarks on the OTLP exporter

In my situation I've deployed my collector in the cloud, but I had some trouble connecting to it. I noticed that data was being sent towards the collector, but it didn't ended up there. After some digging into Wireshark I saw that mu deployed collector returned a 404 Error status code. After digging around for a solution I got it working by adding the correct path and the correct protocol.

Path for the OTLP Exporter 


| Syntax    | Description |
| --------- | ----------- |
| Header    | Title       |
| Paragraph | Text        |


| Signal | Path | Example | 
| -------| -----| --------|
| Metric | v1/metrics | http://mydeployedcollector.com:4318/v1/metrics | 
| Logs | v1/logs | http://mydeployedcollector.com:4318/v1/logs | 
| Traces | v1/traces | http://mydeployedcollector.com:4318/v1/traces | 


````
.AddOtlpExporter(oltpexporter =>
{
        oltpexporter.Endpoint = new System.Uri($"{_configuration["OpenTelemetry:EndPoint"]}/v1/traces");
        oltpexporter.Protocol = OpenTelemetry.Exporter.OtlpExportProtocol.HttpProtobuf;
});

````

With this configuration you will be able to use opentelemetry and by doing so, getting more and more insights into your application. Which might lead to faster bug resolutions, less issues, ... 

Thank you for reading with me. You have a comment or found an issue? Ping me on twitter or leave them over [here](https://github.com/robvanpamel/robvanpamel.github.io/issues/new) 


# References
Some resources which aren't shared yet are mentioned over here. 

- https://opentelemetry.io/docs/collector/
- https://github.com/open-telemetry/opentelemetry-collector/blob/main/receiver/otlpreceiver/README.md 
- https://opentelemetry.io/docs/collector/configuration/#processors