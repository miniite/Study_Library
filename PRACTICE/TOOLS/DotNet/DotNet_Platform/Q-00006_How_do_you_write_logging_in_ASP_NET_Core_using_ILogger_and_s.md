---
type: "practice"
difficulty: "Basic"
---

# How do you write logging in ASP.NET Core using ILogger and structured logging?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Writing logging in ASP.NET Core with ILogger involves setting up a logger factory, injecting it into your services or controllers, and using a structured logging pattern to format log messages. This approach ensures that logs are consistent and can be easily parsed by tools like ELK Stack.

#### Triaging the Question

The question is asking about implementing logging in an ASP.NET Core application using ILogger. The interviewer might be probing on understanding of dependency injection, logging patterns, and structured logging practices. We will assume you have access to a basic ASP.NET Core project setup.

Step-by-Step Solution from First Principles:

Setting Up the Logger Factory:

In your Program.cs or Startup.cs, configure the logger factory.
CSHARP
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Configure logging
        builder.Host.UseSerilog()
                   .ConfigureLogging((context, logging) =>
                   {
                       logging.AddConsole();
                       logging.AddDebug();
                       logging.AddEventSourceLogger();
                   });

        var app = builder.Build();

        if (app.Environment.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.Run();
    }
}

Injecting ILogger into Services or Controllers:

When creating a service, inject the ILogger<T> type and use it to log messages.
CSHARP
public class MyService : IMyService
{
    private readonly ILogger<MyService> _logger;

    public MyService(ILogger<MyService> logger)
    {
        _logger = logger;
    }

    public void DoSomething()
    {
        _logger.LogInformation("Doing something...");
        // Business logic here
    }
}

Using Structured Logging:

To ensure that logs are structured, you can use properties to add context.
CSHARP
_logger.LogWarning(new EventId(100), "Error occurred", new { RequestId = "123456789" });

#### Closing Summary
and Common Mistakes/Follow-Ups:
Summary: Structured logging in ASP.NET Core using ILogger involves configuring the logger factory, injecting it into services or controllers, and utilizing properties to add structured context to log messages.

#### Common Mistakes
Not properly configuring the logger in Program.cs or Startup.cs.
Failing to inject ILogger into classes that need logging.
Overlooking structured logging patterns which can make logs harder to parse and analyze.

If asked follow-ups, you might discuss integration with external log management tools like ELK Stack, or how to handle different log levels (info, warning, error).

**Code:**

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Configure logging
        builder.Host.UseSerilog()
                   .ConfigureLogging((context, logging) =>
                   {
                       logging.AddConsole();
                       logging.AddDebug();
                       logging.AddEventSourceLogger();
                   });

        var app = builder.Build();

        if (app.Environment.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.Run();
    }
}

public class MyService : IMyService
{
    private readonly ILogger<MyService> _logger;

    public MyService(ILogger<MyService> logger)
    {
        _logger = logger;
    }

    public void DoSomething()
    {
        _logger.LogInformation("Doing something...");
        // Business logic here
    }
}
```

**Flow:**

1. Start
2. Configure logger inProgram.cs
3. Create service with ILoggerinjection
4. Log using structured pattern
5. End

