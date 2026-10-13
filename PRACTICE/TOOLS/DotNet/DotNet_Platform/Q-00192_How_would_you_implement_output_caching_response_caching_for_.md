---
type: "practice"
difficulty: "Deep"
---

# How would you implement output caching / response caching for a .NET API considering per-user variance, cache keys with Vary headers, and invalidation when underlying data changes?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement output caching for a .NET API while handling per-user variance, cache keys based on Vary headers, and ensuring proper invalidation when underlying data changes, we need to focus on several key areas: understanding the problem domain, designing an effective caching strategy, implementing the necessary components in .NET, and ensuring robust invalidation mechanisms.

The core challenge here is to create a system that can store and retrieve responses based on various cache keys derived from user inputs or HTTP headers while managing per-user variance and ensuring data consistency. The solution involves leveraging ASP.NET Core's built-in caching capabilities along with custom logic for handling Vary headers and invalidation rules.

**Comparison:**

```
User Id	Vary Headers	Cache Key
user123	Accept-Language, Cookie	user123_Accept-Language_0, Accept-Language__Cookie_
```

**Code:**

```

// Example of setting up response caching in an ASP.NET Core application using Microsoft.AspNetCore.Builder; using Microsoft.Extensions.DependencyInjection;

public class Startup { public void ConfigureServices(IServiceCollection services) { // Register caching services services.AddResponseCaching(); }

CODE
Copy
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers().RequireCors("AllowSpecificOrigin");
        endpoints.EnableEndpointRouting(false);
    });

    // Configure response caching
    app.UseResponseCaching(new ResponseCachingOptions
    {
        Enabled = true,
        LocationHeaderName = "Vary",
        CacheLocation = ResponseCacheLocation.Any,
        MaxAge = TimeSpan.FromMinutes(5)
    });
}

}

CODE
Copy

```csharp
// Example of customizing the response cache key based on Vary headers
using Microsoft.AspNetCore.Http;
using System.Collections.Generic;

public class CustomResponseCachingMiddleware
{
    private readonly RequestDelegate _next;

    public CustomResponseCachingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Capture Vary headers to create a unique cache key
        var varyHeaders = new Dictionary<string, List<string>>();
        foreach (var header in context.Request.Headers)
        {
            if (header.Key.ToLower() == "vary")
            {
                varyHeaders[header.Key] = header.Value.ToList();
            }
        }

        // Generate a unique cache key based on Vary headers and user data
        var cacheKey = $"{context.User.Identity.Name}_{string.Join("_", varyHeaders.Values)}";

        // Check if the response is cached for this request
        context.Response.Cache.SetCacheability(HttpCacheability.Public);
        context.Response.Cache.SetMaxAge(TimeSpan.FromMinutes(5));
        context.Response.Cache.VaryByCustom["VaryHeaders"];

        await _next(context);
    }
}
CSHARP
Copy
// Example of handling data invalidation in a service layer
using Microsoft.Extensions.DependencyInjection;
using System.Threading.Tasks;

public class DataCacheService : IDataCacheService
{
    private readonly IHttpClientFactory _httpClientFactory;

    public DataCacheService(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    public async Task<bool> InvalidateDataCacheAsync(string cacheKey)
    {
        // Simulate invalidation logic using HTTP requests to backend services
        var client = _httpClientFactory.CreateClient("BackendApi");

        try
        {
            await client.GetAsync($"api/data/{cacheKey}");
            return true;
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Failed to invalidate cache for {cacheKey}: {ex.Message}");
            return false;
        }
    }
}
CSHARP
Copy
// Example of integrating caching into a controller action
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;

[Route("api/[controller]")]
[ApiController]
public class DataController : ControllerBase
{
    private readonly IDataCacheService _dataCacheService;

    public DataController(IDataCacheService dataCacheService)
    {
        _dataCacheService = dataCacheService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetDataAsync(string id)
    {
        var cacheKey = $"{id}_{DateTime.UtcNow:yyyyMMdd_HHmmss}";

        // Check if the response is cached
        HttpContext.Response.Cache.SetCacheability(HttpCacheability.Public);
        HttpContext.Response.Cache.SetMaxAge(TimeSpan.FromMinutes(5));
        HttpContext.Response.Cache.VaryByCustom["VaryHeaders"];

        var cachedResult = await HttpContext.GetCachedResponseAsync(cacheKey, "VaryHeaders");
        if (cachedResult != null)
        {
            return Content(cachedResult.Content, "application/json", cachedResult.ContentType);
        }

        // Fetch data from the database or backend
        var data = await GetDataFromDatabaseOrBackendAsync(id);

        // Cache the result
        HttpContext.Response.Cache.SetCacheability(HttpCacheability.Public);
        HttpContext.Response.Cache.SetMaxAge(TimeSpan.FromMinutes(5));
        HttpContext.Response.Cache.VaryByCustom["VaryHeaders"];

        var cacheResult = new CachedResponse(data, "application/json", DateTime.UtcNow.AddMinutes(5), "VaryHeaders");
        await HttpContext.SetCachedResponseAsync(cacheKey, cacheResult);

        return Content(data, "application/json");
    }
}
CSHARP
Copy
// Example of custom `CachedResponse` class for storing and managing cached data
public class CachedResponse
{
    public string Content { get; set; }
    public string ContentType { get; set; }
    public DateTime ExpirationTime { get; set; }
    public string VaryByCustom { get; set; }

    public CachedResponse(string content, string contentType, DateTime expirationTime, string varyByCustom)
    {
        Content = content;
        ContentType = contentType;
        ExpirationTime = expirationTime;
        VaryByCustom = varyByCustom;
    }
}
CSHARP
Copy
// Example of custom `ResponseCachingMiddleware` to handle per-user variance and Vary headers
public class CustomResponseCachingMiddleware
{
    private readonly RequestDelegate _next;

    public CustomResponseCachingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Capture Vary headers to create a unique cache key
        var varyHeaders = new Dictionary<string, List<string>>();
        foreach (var header in context.Request.Headers)
        {
            if (header.Key.ToLower() == "vary")
            {
                varyHeaders[header.Key] = header.Value.ToList();
            }
        }

        // Generate a unique cache key based on Vary headers and user data
        var cacheKey = $"{context.User.Identity.Name}_{string.Join("_", varyHeaders.Values)}";

        // Check if the response is cached for this request
        context.Response.Cache.SetCacheability(HttpCacheability.Public);
        context.Response.Cache.SetMaxAge(TimeSpan.FromMinutes(5));
        context.Response.Cache.VaryByCustom["VaryHeaders"];

        await _next(context);
    }
}
```

**Example input / output:**

```text
Input:
- User Id: user123
- Vary Headers: Accept-Language, Cookie

Output:
- Cache Key: user123_Accept-Language_0, Accept-Language__Cookie_
- Caching Enabled with Max Age of 5 minutes and VaryByCustom set to "VaryHeaders"
```

**Data:**

```json
{
  "User": {
    "Id": "user123",
    "Name": "John Doe"
  },
  "RequestHeaders": {
    "Accept": "*/*",
    "Vary": ["Accept-Language", "Cookie"]
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Cache Enabled?
5. Retrieve From Cache
6. Fetch Data from Backend
7. Cache Response
8. Return Cached Response

