---
type: "practice"
difficulty: "Deep"
---

# Explain how IOptionsMonitor change notifications work and how you'd hot-reload a feature flag or connection setting in a running .NET service without restarting or causing torn reads.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Configuration and Options Pattern](<../../../../TOOLS/DotNet/DotNet_Platform/Configuration_and_Options_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

IOptionsMonitor<T> is a key component of ASP.NET Core that allows you to monitor changes to settings and features such as connection strings or feature flags in real-time, without needing to restart your application. This mechanism ensures that the application can reload configurations when they change, providing seamless hot-reloading capabilities.

#### Triaging the Question

The question asks for an explanation of how IOptionsMonitor<T> works and how it can be used to implement hot-reloading of feature flags or connection settings in a running .NET service. The core of this question is about understanding the underlying mechanism for handling configuration changes dynamically and ensuring that these changes are reflected correctly without causing issues like torn reads.

#### First-Principles Solution
Understanding IOptionsMonitor<T>

IOptionsMonitor<T> is part of ASP.NET Core's dependency injection framework, specifically designed to monitor options (settings) defined in the application’s configuration. When a change occurs in these settings, it notifies all registered services that depend on those settings.

Configuration Source: The settings are read from appsettings.json, environment variables, or other configuration sources.
Dependency Injection: These settings are injected into your classes using dependency injection.
Monitoring Changes: IOptionsMonitor<T> keeps track of these settings and notifies subscribers when changes occur.
Hot-Reloading Feature Flags or Connection Settings

To implement hot-reloading for feature flags or connection settings, follow these steps:

Define the Option Class:

CSHARP
public class MySettings : IOptions<MySettings>
{
    public string ConnectionString { get; set; }
    public bool IsFeatureEnabled { get; set; }

    // Implementing IOptions<T> interface
    public void Set(MySettings value) => this = value;
    public ValueTuple<MySettings, Action<MySettings>> Get() => Tuple.Create(this, _ => { });
}

Register the Option with IOptionsMonitor<T>:

CSHARP
services.Configure<MySettings>(Configuration.GetSection("MySettings"));

Inject and Monitor Changes in Your Service:

CSHARP
public class MyService : IMyService
{
    private readonly MySettings _settings;
    private IDisposable _changeToken;

    public MyService(IOptionsMonitor<MySettings> settings)
    {
        _settings = settings.CurrentValue; // Initial value

        // Subscribe to change notifications
        _changeToken = settings.OnChange(() =>
        {
            // Handle the new configuration values

#### Cons
ole.WriteLine("Configuration changed. New settings: " + _settings.ConnectionString);
        });
    }

    public void DoSomething()
    {
        if (_settings.IsFeatureEnabled)
        {
            // Use updated connection string or feature flag value
        }
    }

    public void Dispose()
    {
        _changeToken.Dispose();
    }
}

Handling Torn Reads:

Ensure that your service logic is thread-safe and handles configuration changes atomically.
Use read-write locks if necessary to prevent torn reads, ensuring consistency during the change transition.
Full Example Code
CSHARP
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Options;

public class MySettings : IOptions<MySettings>
{
    public string ConnectionString { get; set; }
    public bool IsFeatureEnabled { get; set; }

    public void Set(MySettings value) => this = value;
    public ValueTuple<MySettings, Action<MySettings>> Get() => Tuple.Create(this, _ => { });
}

public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<MySettings>(Configuration.GetSection("MySettings"));
        services.AddSingleton<IOptionsMonitor<MySettings>, OptionsMonitor<MySettings>>();
        services.AddTransient<IMyService, MyService>();
    }
}

public class MyService : IMyService
{
    private readonly MySettings _settings;
    private IDisposable _changeToken;

    public MyService(IOptionsMonitor<MySettings> settings)
    {
        _settings = settings.CurrentValue; // Initial value

        // Subscribe to change notifications
        _changeToken = settings.OnChange(() =>
        {

#### Cons
ole.WriteLine("Configuration changed. New settings: " + _settings.ConnectionString);
        });
    }

    public void DoSomething()
    {
        if (_settings.IsFeatureEnabled)
        {
            // Use updated connection string or feature flag value
        }
    }

    public void Dispose()
    {
        _changeToken.Dispose();
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

Implementing hot-reloading with IOptionsMonitor<T> in .NET is a robust way to ensure that your application can handle dynamic configuration changes without restarts. Key aspects include defining the correct option class, properly registering it, handling change notifications, and ensuring thread safety.

Common Pitfalls:

Torn Reads: Not handling changes atomically can lead to inconsistent states.
Memory Leaks: Not disposing of IDisposable tokens can cause memory leaks.
Race Conditions: Improperly handled concurrent access can result in race conditions during configuration transitions.
Final Thoughts

Hot-reloading is a powerful feature that enhances the development experience and reduces downtime. By carefully managing your options and ensuring thread safety, you can leverage this mechanism effectively.

**Code:**

```csharp
public class MySettings : IOptions<MySettings>
{
    public string ConnectionString { get; set; }
    public bool IsFeatureEnabled { get; set; }

    public void Set(MySettings value) => this = value;
    public ValueTuple<MySettings, Action<MySettings>> Get() => Tuple.Create(this, _ => { });
}

public class MyService : IMyService
{
    private readonly MySettings _settings;
    private IDisposable _changeToken;

    public MyService(IOptionsMonitor<MySettings> settings)
    {
        _settings = settings.CurrentValue; // Initial value

        // Subscribe to change notifications
        _changeToken = settings.OnChange(() =>
        {
            Console.WriteLine("Configuration changed. New settings: " + _settings.ConnectionString);
        });
    }

    public void DoSomething()
    {
        if (_settings.IsFeatureEnabled)
        {
            // Use updated connection string or feature flag value
        }
    }

    public void Dispose()
    {
        _changeToken.Dispose();
    }
}
```

**Data:**

```json
{
  "MySettings": {
    "ConnectionString": "new-connection-string",
    "IsFeatureEnabled": true
  }
}
```

**Flow:**

1. Define MySettings class
2. Configure service with options
3. Register IOptionsMonitor
4. Subscribe to changenotifications in serviceconstructor
5. Handle configuration changesin OnChange callback

