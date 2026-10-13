---
type: "practice"
difficulty: "Deep"
---

# How would you add health checks and readiness/liveness probes to an ASP.NET Core service for Kubernetes?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add health checks and readiness/liveness probes to an ASP.NET Core service running in a Kubernetes environment, we need to ensure our application can communicate its state accurately to Kubernetes so that it can make informed decisions about the pod’s lifecycle—such as restarting or scaling up. This involves implementing liveness and readiness probes at both the application level and leveraging built-in monitoring features provided by Kubernetes.

#### Triage

The key aspects of the question are:

Liveness Probes: Determine if your application is still running.
Readiness Probes: Determine if your application can handle incoming traffic.
Health Checks: Ensure that the service’s internal state is healthy and it’s ready to serve requests.

We will use Kubernetes’ livenessProbe and readinessProbe mechanisms, which can be configured via annotations in the Deployment or by customizing the ASP.NET Core application itself.

#### First-Principles Solution

Liveness Probes:

These are used to determine if an application has crashed or is unresponsive.
We should periodically send a request to a specific endpoint (e.g., /liveness) and check for a successful response within the defined grace period.

Readiness Probes:

These ensure that an application can handle incoming traffic without causing any issues.
Typically, we would check if the service is ready to accept requests by sending another type of request (e.g., /readiness).

Health Checks in ASP.NET Core:

Implement these checks as middleware or use a custom HTTP client to make health check requests from within the application.
Detailed Implementation

To implement liveness and readiness probes, we can leverage Kubernetes annotations directly or create an internal endpoint that performs the necessary checks:

Using Annotations (Recommended)

In your Kubernetes Deployment YAML file:

YAML
spec:
  template:
    spec:
      containers:
        - name: my-app
          image: my-app-image
          livenessProbe:
            httpGet:
              path: /liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
Internal Endpoint Implementation in ASP.NET Core

Create a simple health check controller:

CSHARP
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace MyNamespace.Controllers
{
    [Route("api/[controller]")]
    public class HealthCheckController : Controller
    {
        private readonly IHealthChecker _healthChecker;

        public HealthCheckController(IHealthChecker healthChecker)
        {
            _healthChecker = healthChecker;
        }

        /// <summary>
        /// Liveness Probe Endpoint
        /// </summary>
        [HttpGet("/liveness")]
        public IActionResult Liveness()
        {
            return Ok(_healthChecker.CheckLiveness());
        }

        /// <summary>
        /// Readiness Probe Endpoint
        /// </summary>
        [HttpGet("/readiness")]
        public IActionResult Readiness()
        {
            return Ok(_healthChecker.CheckReadiness());
        }
    }
}
Health Checker Implementation
CSHARP
using System;
using Microsoft.Extensions.Hosting;

public class HealthChecker : IHealthChecker, IDisposable
{
    private readonly IHostApplicationLifetime _appLifetime;

    public HealthChecker(IHostApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public bool CheckLiveness()
    {
        // Simple health check to ensure the application is running.
        try
        {
            return true; // Application is up and running.
        }
        catch (Exception ex)
        {

#### Cons
ole.WriteLine($"Health Check Error: {ex.Message}");
            return false;
        }
    }

    public bool CheckReadiness()
    {
        // More complex check to ensure the service can handle requests.
        try
        {
            // Perform necessary checks, e.g., database connection, etc.
            // For simplicity, assume all is well if liveness is true.
            return this.CheckLiveness();
        }
        catch (Exception ex)
        {

#### Cons
ole.WriteLine($"Readiness Check Error: {ex.Message}");
            return false;
        }
    }

    public void Dispose()
    {
        _appLifetime.StopApplication(); // Graceful shutdown
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing health checks and probes as described, we ensure that the ASP.NET Core application can communicate its state accurately to Kubernetes. This setup helps in maintaining a robust and resilient service by automatically restarting unresponsive containers or scaling them based on readiness.

#### Common Mistakes

Failing to define clear and distinct endpoints for liveness and readiness.
Not configuring initial delays correctly, leading to unnecessary restarts.
Missing appropriate error handling and logging.

#### Follow-ups
could include:

Discussing how to integrate these probes with custom monitoring tools (e.g., Prometheus).
Explaining the impact of different probe configurations on resource utilization and application performance.
Deeper discussion on advanced health checks involving complex state checks beyond simple HTTP GET requests.

**Comparison:**

```
Probe Type	Endpoint	Initial Delay (s)	Periodicity (s)
Liveness	/liveness	30	10
Readiness	/readiness	5	10
```

**Code:**

```

using Microsoft.AspNetCore.Http; using Microsoft.AspNetCore.Mvc;

namespace MyNamespace.Controllers { [Route("api/[controller]")] public class HealthCheckController : Controller { private readonly IHealthChecker _healthChecker;

CODE
Copy
    public HealthCheckController(IHealthChecker healthChecker)
    {
        _healthChecker = healthChecker;
    }

    /// <summary>
    /// Liveness Probe Endpoint
    /// </summary>
    [HttpGet("/liveness")]
    public IActionResult Liveness()
    {
        return Ok(_healthChecker.CheckLiveness());
    }

    /// <summary>
    /// Readiness Probe Endpoint
    /// </summary>
    [HttpGet("/readiness")]
    public IActionResult Readiness()
    {
        return Ok(_healthChecker.CheckReadiness());
    }
}

}

CODE
Copy

```csharp
using System;
using Microsoft.Extensions.Hosting;

public class HealthChecker : IHealthChecker, IDisposable
{
    private readonly IHostApplicationLifetime _appLifetime;

    public HealthChecker(IHostApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public bool CheckLiveness()
    {
        try
        {
            return true; // Application is up and running.
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Health Check Error: {ex.Message}");
            return false;
        }
    }

    public bool CheckReadiness()
    {
        try
        {
            // Perform necessary checks, e.g., database connection, etc.
            // For simplicity, assume all is well if liveness is true.
            return this.CheckLiveness();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Readiness Check Error: {ex.Message}");
            return false;
        }
    }

    public void Dispose()
    {
        _appLifetime.StopApplication(); // Graceful shutdown
    }
}
```

**Example input / output:**

```text
# Example input for Kubernetes Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app-image
          livenessProbe:
            httpGet:
              path: /liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

**Data:**

```yaml
spec:
  template:
    spec:
      containers:
        - name: my-app
          image: my-app-image
          livenessProbe:
            httpGet:
              path: /liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

**Flow:**

1. Yes
2. No
3. Health Check Request
4. Is Application Running?
5. Return OK
6. Log Error and Fail HealthCheck
7. Application Restart or ScalingAction

