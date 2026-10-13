---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between AddSingleton, AddScoped, and AddTransient service lifetimes?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In .NET, understanding how services are scoped (lifecycle) within an application is crucial for managing resources effectively and ensuring your code behaves as expected. The three primary service lifetimes in .NET are AddSingleton, AddScoped, and AddTransient. Let's break down each of these concepts from first principles to understand the differences between them.

#### Triage

The question asks us to explain the differences between AddSingleton, AddScoped, and AddTransient when setting up services in a .NET application. The interviewer is probing your understanding of service lifetime management, which directly impacts how resources are managed within your application.

#### Assumptions
You have basic knowledge of dependency injection (DI) in .NET.
You understand that each service lifetime controls the creation and disposal of instances of services throughout the application's lifecycle.

#### First-Principles Solution
AddSingleton: This is a global singleton instance per Application Domain. The service will be created only once during the application’s lifetime, which makes it ideal for caching or other long-lived scenarios where you want to share the same instance across all requests.
AddScoped: A scoped instance of the service is created per request (or more specifically, per scope). This means that a new instance of the service will be created every time a specific request starts and disposed when the request ends. It's commonly used for things like database contexts or user-specific data.
AddTransient: A transient instance of the service is created each time it’s resolved from the DI container. This means a fresh copy of the service is instantiated every single time, making it suitable for scenarios where you don’t want to maintain any state between requests.

#### Closing Summary
and Common Mistakes

Understanding these lifetimes correctly ensures that resources are managed efficiently and avoids memory leaks or unexpected behaviors in your application. Common mistakes include not properly understanding when to use each lifetime, leading to issues like service instances being shared across requests where they shouldn't be (e.g., using AddScoped for a global cache).

#### Example
CSHARP
// AddSingleton example
services.AddSingleton<IMyService, MyService>();

// AddScoped example
services.AddScoped<IMyService, MyService>();

// AddTransient example
services.AddTransient<IMyService, MyService>();
Common Follow-Ups
What happens if you mix these lifetimes in the same application?
How does this impact performance and resource management?

**Code:**

```csharp
// Example of registering services with different lifetimes
public void ConfigureServices(IServiceCollection services)
{
    // AddSingleton example
    services.AddSingleton<IMyService, MyService>();

    // AddScoped example
    services.AddScoped<IMyService, MyService>();

    // AddTransient example
    services.AddTransient<IMyService, MyService>();
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[AddSingleton] --> B{Is the service shared across all requests?};
4. B -- Yes --> C[Yes, single instance per App Domain];
5. B -- No --> D{Is the service request-scoped?};
6. D -- Yes --> E[Yes, new instance per request];
7. D -- No --> F[New instance each time it is resolved];
8. graph LR;
9. A1[AddSingleton] --> B1{Is the service shared across all requests?};
10. B1 -- Yes --> C1[Yes, single instance per App Domain];
11. B1 -- No --> D1{Is the service request-scoped?};
12. D1 -- Yes --> E1[Yes, new instance per request];
13. D1 -- No --> F1[New instance each time it is resolved];
14. graph TB;
15. A2[AddSingleton] --> B2{Is the service shared across all requests?};
16. B2 -- Yes --> C2[Yes, single instance per App Domain];
17. B2 -- No --> D2{Is the service request-scoped?};
18. D2 -- Yes --> E2[Yes, new instance per request];
19. D2 -- No --> F2[New instance each time it is resolved];

