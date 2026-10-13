---
type: "practice"
difficulty: "Intermediate"
---

# How do you safely deprecate and remove an endpoint consumed by unknown internal clients — what telemetry, sunset headers, and rollout gating do you put in place first?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deprecating or removing a .NET backend endpoint that is being used by unknown internal clients, it's crucial to ensure a smooth transition without causing service disruptions. The key steps involve setting up telemetry, using sunset headers, and implementing careful rollout gating.

Firstly, understanding the context of your internal clients is vital. These could be various microservices or applications running in different environments, possibly across multiple teams. By tracking how many requests are coming from each client, you can identify critical dependencies that need more attention during the deprecation process.

To start, set up telemetry to monitor endpoint usage:

CSHARP
// Example of setting up performance counters for tracking request counts
using System.Diagnostics;

public class EndpointTelemetry
{
    private static PerformanceCounter _requestsPerSec;

    public static void Initialize()
    {
        _requestsPerSec = new PerformanceCounter("MyApp", "Requests per Sec", "EndpointName");
        _requestsPerSec.EnablePerformanceCounters();
    }
}

Next, implement sunset headers in the HTTP responses to inform clients about impending changes:

HTTP
HTTP/1.1 200 OK
X-Deprecated: true
Content-Type: application/json

{
  "message": "This endpoint will be deprecated soon."
}

These headers should include a specific date when the endpoint will fully shut down, giving internal teams enough time to migrate their code.

Lastly, rollout gating is essential to control the spread of deprecation. This can be achieved through feature flags or conditional logic based on client metadata:

CSHARP
public async Task<IActionResult> EndpointName()
{
    var isDeprecated = await _featureToggleService.IsFeatureEnabled("endpoint-deprecation");

    if (isDeprecated)
    {
        return StatusCode(StatusCodes.Status410 Gone, new { message = "This endpoint has been deprecated." });
    }

    // Existing logic to handle requests
}

**Code:**

```csharp
// Example of setting up performance counters for tracking request counts
using System.Diagnostics;

public class EndpointTelemetry
{
    private static PerformanceCounter _requestsPerSec;

    public static void Initialize()
    {
        _requestsPerSec = new PerformanceCounter("MyApp", "Requests per Sec", "EndpointName");
        _requestsPerSec.EnablePerformanceCounters();
    }
}

// Example of implementing sunset headers in .NET
public class EndpointController : ControllerBase
{
    private readonly IFeatureToggleService _featureToggleService;

    public EndpointController(IFeatureToggleService featureToggleService)
    {
        _featureToggleService = featureToggleService;
    }

    [HttpGet("endpoint")]
    public async Task<IActionResult> EndpointName()
    {
        var isDeprecated = await _featureToggleService.IsFeatureEnabled("endpoint-deprecation");

        if (isDeprecated)
        {
            Response.Headers.Add("X-Deprecated", "true");
            return StatusCode(StatusCodes.Status410 Gone, new { message = "This endpoint has been deprecated." });
        }

        // Existing logic to handle requests
        return Ok(new { data = "Endpoint response" });
    }
}
```

**Example input / output:**

```text
// Before sunset headers are enabled
GET /endpoint HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": "Endpoint response"
}

// After sunset headers are enabled
GET /endpoint HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
X-Deprecated: true
Content-Type: application/json

{
  "message": "This endpoint will be deprecated soon."
}
```

**Flow:**

1. Yes
2. No
3. Deprecation Start
4. Is Feature Enabled?
5. Send Sunset Headers
6. Handle Requests Normally
7. Monitor Usage with Telemetry

