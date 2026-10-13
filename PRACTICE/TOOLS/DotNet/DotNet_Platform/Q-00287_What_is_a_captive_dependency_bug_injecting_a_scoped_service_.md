---
type: "practice"
difficulty: "Deep"
---

# What is a captive dependency bug (injecting a scoped service into a singleton) and why is it dangerous?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Thread Safety and Locking](<../../../../TOOLS/DotNet/DotNet_Platform/Thread_Safety_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A captive dependency bug occurs when you inject a scoped service into a singleton service in .NET, which can lead to subtle bugs that are difficult to trace. The term "captive" refers to how the scoped service is effectively captured and held onto by the singleton, causing it to behave unexpectedly. This issue is particularly dangerous because it often manifests only under specific conditions or environments, making it challenging to identify during testing.

#### Triage
Understanding the Question

The question asks us to explain what a captive dependency bug is and why it's problematic. The key concepts here are:

Scoped Services: These services are created per HTTP request in ASP.NET Core.
Singleton Services: These services have a single instance that is shared across all requests.
Dependency Injection (DI): This design pattern is used to manage the dependencies of an application.
Step-by-Step Solution
1. Definition and Example

In .NET, when you inject a scoped service into a singleton, the singleton retains a reference to this scoped service instance throughout its lifetime. Here's an example:

CSHARP
public class SingletonClass
{
    private readonly IScopedService _scopedService;

    public SingletonClass(IScopedService scopedService)
    {
        _scopedService = scopedService;
    }

    // Method that uses the scoped service
    public void DoSomething()
    {
        // Business logic using _scopedService
        _scopedService.SomeMethod();
    }
}

In this case, _scopedService is a scoped service injected into SingletonClass. However, because SingletonClass is a singleton, it keeps a reference to the scoped instance of IScopedService.

2. Why It's Dangerous

The danger lies in the fact that the scoped service can outlive its intended scope, leading to issues such as:

State Accumulation: The scoped service might accumulate state across multiple HTTP requests due to being referenced by a singleton.
Incorrect Behavior: Actions performed on the scoped service within one request may affect the behavior of subsequent requests, making the application behave unpredictably.
3. Consequences

A common consequence of this bug is:

Race Conditions: If two different requests interact with the same scoped instance through the singleton, they might interfere with each other's state.
Testing Difficulties: This issue can be hard to detect in unit tests because it often only manifests under specific runtime conditions.

#### Closing Summary
and Common Mistakes

To avoid captive dependency bugs:

Use Singleton Services Wisely: Ensure that a scoped service is not injected into a singleton unless absolutely necessary.
Test Thoroughly: Run integration tests across different request scenarios to catch these issues early.
Review Dependency Injection Patterns: Regularly review your DI setup for any potential captive dependencies.

#### Common Mistakes
Misunderstanding Service Lifetime: Failing to properly understand the lifecycle differences between scoped and singleton services.
Overusing Singleton Services: Assuming that a service needs to be singleton when it actually should be scoped or transient.
Inadequate Testing Coverage: Not having comprehensive test cases that cover various request sequences.

#### Conclusion

Understanding captive dependency bugs is crucial for developing robust applications in .NET. By recognizing the potential pitfalls and taking proactive steps to avoid them, you can significantly improve the reliability and maintainability of your codebase.

**Code:**

```csharp
public class SingletonClass
{
    private readonly IScopedService _scopedService;

    public SingletonClass(IScopedService scopedService)
    {
        _scopedService = scopedService;
    }

    // Method that uses the scoped service
    public void DoSomething()
    {
        // Business logic using _scopedService
        _scopedService.SomeMethod();
    }
}
```

