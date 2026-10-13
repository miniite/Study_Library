---
type: "practice"
difficulty: "Deep"
---

# What is OpenTelemetry in .NET and how would you export traces and metrics from an ASP.NET Core service?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Configuration and Options Pattern](<../../../../TOOLS/DotNet/DotNet_Platform/Configuration_and_Options_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

OpenTelemetry (OTel) is a comprehensive observability framework that helps developers collect, send, and analyze telemetry data such as trace data, metrics, and logs from their applications. In the context of an ASP.NET Core service, OTel allows you to instrument your application to capture detailed information about its behavior and performance.

#### Triaging the Question

The question is essentially asking two things:

What is OpenTelemetry in .NET?
How would you export traces and metrics from an ASP.NET Core service using OpenTelemetry?

#### Assumptions

You are familiar with basic concepts of telemetry data, such as traces and metrics.
The environment involves a .NET application (specifically ASP.NET Core), and the goal is to integrate OTel for observability.
Step-by-Step Solution
1. Understanding OpenTelemetry in .NET

OpenTelemetry consists of two primary components:

Collector: A centralized service that collects telemetry data from multiple sources.
SDKs: Libraries that allow you to instrument your application code, injecting tracing and metric collection at the source.

In .NET, OTel is implemented through various SDKs and libraries. The key libraries include OpenTelemetry, OpenTelemetry.Exporter, and specific exporters for different backends like Jaeger or Prometheus.

2. Setting Up OpenTelemetry in an ASP.NET Core Service

To export traces and metrics from an ASP.NET Core service, you need to follow these steps:

Install the necessary NuGet packages:
SH
dotnet add package OpenTelemetry.AspNetCore
dotnet add package OpenTelemetry.Exporter.Jaeger
Configure the OpenTelemetry SDK in Program.cs or Startup.cs:
CSHARP
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;
using OpenTelemetry.Exporter;

public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Set up the OpenTelemetry SDK
        builder.Services.AddOpenTelemetry()
            .WithTracing(tracerProvider =>
                tracerProvider.SetResourceBuilder(
                    Resource.CreateDefault().AddService("MyApp"))
                    .AddSource("MyApp")
                    .AddAspNetCoreInstrumentation()
                    .AddJaegerExporter(options => options.AgentHost = "localhost"; options.AgentPort = 6831))
            .WithMetrics(meterProvider =>
                meterProvider.AddConfiguration("MyApp", o => { }))
            .Build();

        var app = builder.Build();
        app.UseRouting();
        app.MapGet("/", () => "Hello, OpenTelemetry!");
        app.Run();
    }
}
Instrument your application:
The AddAspNetCoreInstrumentation() method automatically instruments the ASP.NET Core pipeline.
You can use other instrumentation methods as needed (e.g., for database calls).
3. Exporting Traces and Metrics

Traces are collected by the OpenTelemetry SDK and sent to a collector or backend via an exporter. In this example, we’re using Jaeger.

Jaeger Setup:
Ensure that a Jaeger instance is running.
The AddJaegerExporter method in the configuration sets up the exporter with appropriate settings (host and port).
4. Verifying the Setup

You can verify that traces are being collected by navigating to your Jaeger UI or using the Jaeger CLI.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, OpenTelemetry provides a robust framework for collecting and exporting telemetry data from .NET applications like ASP.NET Core services. Proper setup involves installing necessary NuGet packages, configuring the SDK, and setting up exporters to send data to a collector or backend service.

#### Common Mistakes

Forgetting to add the AddOpenTelemetry call in the service configuration.
Incorrectly configuring the Jaeger exporter settings (e.g., wrong host or port).
Not instrumenting parts of your application that you need to monitor.

#### Follow-ups
might include:

Discussing other exporters (Prometheus, OTEL Collector) and their use cases.
Explaining how to configure additional instruments for metrics beyond those provided by ASP.NET Core (e.g., custom metric collections).

**Comparison:**

```
Component	Description
OpenTelemetry.AspNetCore	ASP.NET Core-specific instrumentation
OpenTelemetry.Exporter.Jaeger	Export traces to Jaeger backend
JSON
Copy
{
  "service": "MyApp",
  "instrumentation": [
    {
      "name": "AspNetCoreInstrumentation",
      "type": "Automatic"
    },
    {
      "name": "JaegerExporter",
      "settings": {
        "agentHost": "localhost",
        "agentPort": 6831
      }
    }
  ]
}
PLAINTEXT
Copy
Input:
- Installed NuGet packages `OpenTelemetry.AspNetCore` and `OpenTelemetry.Exporter.Jaeger`.
- Configured OpenTelemetry SDK in `Program.cs`.

Output:
- Traces are sent to Jaeger backend at `localhost:6831`.
```

**Code:**

```csharp
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;
using OpenTelemetry.Exporter;

public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Set up the OpenTelemetry SDK
        builder.Services.AddOpenTelemetry()
            .WithTracing(tracerProvider =>
                tracerProvider.SetResourceBuilder(
                    Resource.CreateDefault().AddService("MyApp"))
                .AddSource("MyApp")
                .AddAspNetCoreInstrumentation()
                .AddJaegerExporter(options => options.AgentHost = "localhost"; options.AgentPort = 6831))
            .WithMetrics(meterProvider =>
                meterProvider.AddConfiguration("MyApp", o => { }))
            .Build();

        var app = builder.Build();
        app.UseRouting();
        app.MapGet("/", () => "Hello, OpenTelemetry!");
        app.Run();
    }
}
Component	Description
OpenTelemetry.AspNetCore	ASP.NET Core-specific instrumentation
OpenTelemetry.Exporter.Jaeger	Export traces to Jaeger backend
```

**Data:**

```json
{
  "service": "MyApp",
  "instrumentation": [
    {
      "name": "AspNetCoreInstrumentation",
      "type": "Automatic"
    },
    {
      "name": "JaegerExporter",
      "settings": {
        "agentHost": "localhost",
        "agentPort": 6831
      }
    }
  ]
}
```

**Flow:**

1. Configure OpenTelemetry SDK
2. Install NuGet packages
3. Set up tracing and metrics inProgram.cs
4. Instrument ASP.NET Corepipeline
5. Start application
6. Navigate to Jaeger UI to verifytraces

