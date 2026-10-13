---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle exceptions globally in ASP.NET Core using exception-handling middleware?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle exceptions globally in an ASP.NET Core application, we use a combination of middleware and global exception handling methods. This approach ensures that any unhandled exceptions are caught and managed centrally, providing consistent error handling throughout the application.

#### Triage

The interviewer is probing for your understanding of how to implement centralized error handling in ASP.NET Core applications. They are likely interested in how you manage exceptions at different stages of request processing, such as during middleware execution or within controllers. Additionally, they might ask about logging, custom error pages, and the impact on performance.

#### First-Principles Solution

In ASP.NET Core, global exception handling is typically achieved by configuring middleware to catch unhandled exceptions. We can use UseExceptionHandler in the application's startup configuration to define a custom error page or handler for production environments. For development, we might want more detailed logging and debugging capabilities.

Step-by-Step Solution

Configure Middleware:

In the Startup.cs file, specifically within the ConfigureServices method, add services that may throw exceptions.
In the Configure method, use UseExceptionHandler to handle unhandled exceptions.

Custom Exception Handling Middleware:

Create a middleware component that will catch and process any unhandled exceptions.
This middleware can log detailed error information and take appropriate actions (e.g., redirecting to an error page).

Global Exception Filter:

Implement global exception filters using attributes or the ApplicationExceptionFilter class, which allows you to handle exceptions at a higher level.

#### Example
Implementation
CSHARP
public void ConfigureServices(IServiceCollection services)
{
    // Register your services here
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseRouting();

    // Custom middleware to log exceptions
    app.UseMiddleware<ExceptionLoggingMiddleware>();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}

public class ExceptionLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public ExceptionLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            // Log the exception

#### Cons
ole.WriteLine($"An unhandled exception occurred: {ex.Message}");
            // Redirect to a custom error page or handle as needed
            context.Response.Redirect("/Error");
        }
    }
}

#### Closing Summary
and Common Mistakes

Implementing global exception handling in ASP.NET Core is crucial for maintaining the stability and reliability of your application. Always ensure that exceptions are logged appropriately, especially in production environments. Common mistakes include not configuring UseExceptionHandler correctly or failing to handle sensitive information like stack traces.

#### Follow-Ups
How do you manage performance impacts when handling exceptions globally?
Can you explain how custom error pages can be configured and displayed?
What steps would you take if an exception occurs during the middleware execution itself?

**Code:**

```csharp
public class ExceptionLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public ExceptionLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An unhandled exception occurred: {ex.Message}");
            context.Response.Redirect("/Error");
        }
    }
}
```

