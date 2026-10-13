---
type: "practice"
difficulty: "Intermediate"
---

# What is content negotiation in ASP.NET Core and how does an API decide between JSON and other formats?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Content negotiation in ASP.NET Core allows APIs to dynamically choose the response format based on client preferences, such as whether the client prefers JSON or another format like XML. This is crucial for building flexible and user-friendly APIs that can serve multiple types of clients.

#### Triage

The question is essentially asking about how ASP.NET Core handles content negotiation between different data formats, particularly focusing on JSON vs other formats. The interviewer is likely probing your understanding of the underlying mechanisms in ASP.NET Core to handle these scenarios effectively. Key assumptions include familiarity with HTTP headers and middleware concepts in ASP.NET Core.

#### First-Principles Solution
Understanding Content Negotiation: Content negotiation happens when a client requests content that can be provided in different formats, such as JSON or XML.
HTTP Headers: The Accept header in the request is crucial for this process. It specifies the client's preferred format (e.g., application/json, application/xml).
Middleware Configuration: ASP.NET Core uses middleware to intercept and handle requests. The default middleware includes an IActionResultSelector which decides how to return a result based on various factors, including content negotiation.
Result Formatters: These are responsible for converting the model data into the desired format before sending it back as a response.
Step-by-Step Solution
HTTP Request Analysis:
The client sends an HTTP request with a Accept header indicating its preference (e.g., application/json).
Middleware Processing:
ASP.NET Core middleware inspects the Accept header to determine the preferred format.
Result Formatting:
If JSON is requested, ASP.NET Core uses the built-in JSON formatter to serialize the data into a JSON response.
For other formats like XML or custom formats, you might need to configure additional formatters.

#### Example

Here’s an example of configuring content negotiation in ASP.NET Core:

CSHARP
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddControllers()
        .AddJsonOptions(options =>
        {
            options.JsonSerializerOptions.PropertyNamingPolicy = null; // Customize JSON serialization as needed
        });

    // Optionally, add XML formatter if needed
    services.Configure<ApiBehaviorOptions>(options =>
    {
        options.SuppressModelStateInvalidFilter = true;
    });

    services.AddControllers();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}

#### Closing Summary
and Common Mistakes

In summary, content negotiation in ASP.NET Core is handled through HTTP headers and middleware that dynamically choose the appropriate formatter based on client preferences. Common mistakes include not properly configuring formatters or misinterpreting the Accept header.

Always ensure you check for common formats like JSON and XML explicitly when setting up your application to handle different content types effectively.

#### Common Mistakes/Follow-Ups
Misunderstanding Accept Headers: Clients might send unexpected or unsupported Accept headers, leading to errors if not handled properly.
Formatter Configuration: Not configuring formatters for all expected formats can lead to runtime exceptions when a client requests an unhandled format.

**Comparison:**

```
Header	Value
Accept	application/json
```

**Code:**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers()
        .AddJsonOptions(options =>
        {
            options.JsonSerializerOptions.PropertyNamingPolicy = null; // Customize JSON serialization as needed
        });

    // Optionally, add XML formatter if needed
    services.Configure<ApiBehaviorOptions>(options =>
    {
        options.SuppressModelStateInvalidFilter = true;
    });

    services.AddControllers();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

**Example input / output:**

```

Request:

CODE
Copy
GET /api/data HTTP/1.1
Host: example.com
Accept: application/json

Response:

CODE
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
    "data": "some data"
}
```

**Data:**

```json
{
  "Accept": "application/json"
}
```

**Flow:**

1. Client Request
2. Read Accept header
3. Select formatter based onAccept
4. Serialize data to JSON/XML
5. Send response withContent-Type set

