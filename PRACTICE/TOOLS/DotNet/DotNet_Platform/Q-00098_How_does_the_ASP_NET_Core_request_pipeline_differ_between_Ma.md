---
type: "practice"
difficulty: "Intermediate"
---

# How does the ASP.NET Core request pipeline differ between Map, Use, and Run terminal middleware?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In ASP.NET Core, Map, Use, and Run are middleware methods that dictate how requests are processed in different stages of the request pipeline. Map is used to route requests based on URL patterns, Use applies middleware globally or conditionally, and Run executes middleware at the very end of the pipeline.

#### Triaging the Question

This question asks about the differences between three types of middleware methods—Map, Use, and Run—in the ASP.NET Core request pipeline. The interviewer is probing your understanding of how these methods fit into the overall pipeline architecture, their unique roles, and when to use each one.

#### First Principles Solution

Understanding Middleware: Middleware in ASP.NET Core are classes that implement the Middleware interface or can be a simple class that has an asynchronous method called Invoke. They process HTTP requests before they reach your application's handlers and can also process the responses before they are sent back to the client.

Map Middleware:

Purpose: Map is used to configure route constraints and middleware based on URL patterns.
Behavior: It does not apply any middleware but instead configures additional routes or middleware for requests that match a specific pattern.

#### Example
You can use Map to specify different handlers for subdomains, paths, or other URL segments.

Use Middleware:

Purpose: Use applies middleware globally or conditionally based on the request's characteristics.
Behavior: It can be used multiple times to apply multiple pieces of middleware in a chain. Each piece of middleware in the pipeline is called sequentially, with each one having the opportunity to modify the request and response objects.

#### Example
Common uses include authentication, logging, and request validation.

Run Middleware:

Purpose: Run executes middleware at the very end of the pipeline, immediately before the controller action or endpoint handler is invoked.
Behavior: It can only be used once per pipeline and is typically used for final processing, such as logging response times or generating a unique request ID.

#### Example
You might use Run to log the time it took to execute an action method.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, Map, Use, and Run are used at different stages of the request pipeline with distinct purposes. Map is for configuring routes, Use is for applying middleware conditionally or globally, and Run is for executing final processing before reaching the action method.

#### Common mistakes
include:

Misusing Use in a way that prevents further middleware from being applied.
Overusing Run, which can clutter your pipeline with unnecessary logic.
Not understanding the implications of using Map for complex routing patterns, potentially leading to hard-to-maintain code.

During an interview, you might be asked:

How would you handle complex routing scenarios?
Can you provide an example where misusing these middleware methods could cause issues in your application?

**Code:**

```csharp
// Example of using Map, Use, and Run

app.Use(async (context, next) =>
{
    // Middleware logic here
});

app.Map("/api/v1", apiApp =>
{
    apiApp.Use(async (context, next) =>
    {
        // API-specific middleware
    });

    apiApp.Run(context =>
    {
        // Final processing for this route
    });
});
```

