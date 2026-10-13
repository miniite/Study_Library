---
type: "practice"
difficulty: "Intermediate"
---

# How would you register multiple implementations of an interface and resolve the right one at runtime?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ JIT vs AOT Compilation](<../../../../TOOLS/DotNet/DotNet_Platform/JIT_vs_AOT_Compilation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to register multiple implementations of an interface in .NET and resolve the appropriate implementation at runtime, we need to ensure that our design is flexible and extensible. At its core, this problem involves dependency injection (DI) with a strategy pattern or factory pattern.

The question itself probes your understanding of:

Dependency Injection: How to decouple dependencies from classes.
Factory Pattern: For creating objects based on some external input without specifying the exact class.
Runtime Resolution: Choosing the correct implementation at runtime, possibly based on configuration or context.

Let's reason through this step-by-step:

Step 1: Define the Problem

We have a scenario where we need to work with multiple implementations of an interface, and our application should be able to choose the right one dynamically. For example, consider a Logger interface with different concrete implementations like ConsoleLogger, FileLogger, etc.

CSHARP
public interface ILogger
{
    void Log(string message);
}
Step 2: Implement Concrete Classes

Here are a couple of possible implementations:

CSHARP
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {

#### Cons
ole.WriteLine($"Console Logger: {message}");
    }
}

public class FileLogger : ILogger
{
    private readonly string _filePath;

    public FileLogger(string filePath)
    {
        _filePath = filePath;
    }

    public void Log(string message)
    {
        File.AppendAllText(_filePath, $"File Logger: {message}\n");
    }
}
Step 3: Register Implementations with a DI Container

Using a dependency injection container like Microsoft.Extensions.DependencyInjection, we can register these implementations. Let's assume our application needs to decide which logger to use based on some input parameter.

CSHARP
public void ConfigureServices(IServiceCollection services)
{
    // Register the interfaces and their concrete implementations.
    services.AddTransient<ILogger, ConsoleLogger>();
    services.AddSingleton<ILogger, FileLogger>(sp => new FileLogger("app.log"));
}
Step 4: Resolve Implementations at Runtime

To resolve the appropriate ILogger implementation at runtime, we can use a factory method or a strategy. For example, we might have a configuration setting in our application that determines which logger to use.

CSHARP
public class LoggerFactory
{
    private readonly ILogger _consoleLogger;
    private readonly ILogger _fileLogger;

    public LoggerFactory(ILogger consoleLogger, ILogger fileLogger)
    {
        _consoleLogger = consoleLogger;
        _fileLogger = fileLogger;
    }

    public void Log(string message)
    {
        // Example: Decide based on a config setting
        if (ShouldUseConsole())
        {
            _consoleLogger.Log(message);
        }
        else
        {
            _fileLogger.Log(message);
        }
    }

    private bool ShouldUseConsole()
    {
        // This could be replaced with actual configuration logic.
        return true;
    }
}
Step 5: Use the Factory or Strategy

Finally, we can use the factory to log messages based on our decision criteria.

CSHARP
public class Program
{
    public static void Main(string[] args)
    {
        var serviceProvider = new ServiceCollection()
            .AddTransient<ILogger, ConsoleLogger>()
            .AddSingleton<ILogger, FileLogger>("app.log")
            .BuildServiceProvider();

        var loggerFactory = new LoggerFactory(serviceProvider.GetRequiredService<ILogger>(),
                                             serviceProvider.GetRequiredService<ILogger>());

        loggerFactory.Log("Hello, World!");
    }
}

#### Closing Summary
and Common Mistakes

This solution ensures that our application can dynamically choose the right logging mechanism based on runtime conditions. The key steps include:

Registering multiple implementations of an interface.
Resolving instances at runtime using a factory or strategy pattern.

#### Common mistakes
to avoid:

Failing to properly register different types in the DI container.
Not handling the case where no suitable implementation can be found.
Hardcoding decision logic, which makes it less flexible and harder to maintain.

