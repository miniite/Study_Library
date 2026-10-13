---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a unit test and an integration test, and how do you integration-test an ASP.NET Core API with WebApplicationFactory?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Middleware Pipeline](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Middleware_Pipeline.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with .NET, understanding the differences between unit tests and integration tests is crucial. Unit tests focus on isolated units of code, typically classes or methods, to ensure they function correctly in isolation from other parts of the application. Integration tests, on the other hand, test how different components within an application interact with each other. This distinction is particularly important when it comes to testing ASP.NET Core APIs.

To integrate-test an ASP.NET Core API using WebApplicationFactory, we need a way to start and run the application in memory, allowing us to test various endpoints and functionalities without hitting the database or external services directly. In this answer, I'll explain the difference between unit tests and integration tests and show how to use WebApplicationFactory for integration testing an ASP.NET Core API.

**Comparison:**

```
Test Type	Focus	Scope
Unit Test	Class or method	Isolated units of code in a vacuum.
Integration Test	Components	Interaction between different parts of an application.
```

**Code:**

```csharp
using Xunit;
using System.Net.Http;
using Microsoft.AspNetCore.Mvc.Testing;

public class ExampleApiIntegrationTests : IClassFixture<WebApplicationFactory<Startup>>
{
    private readonly WebApplicationFactory<Startup> _factory;

    public ExampleApiIntegrationTests(WebApplicationFactory<Startup> factory)
    {
        _factory = factory;
    }

    [Fact]
    public async Task Should_Return_Expected_Response()
    {
        // Arrange
        var client = _factory.CreateClient();

        // Act
        var response = await client.GetAsync("/api/example");

        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        Assert.Contains("expected data", content);
    }
}
```

**Example input / output:**

```text
- **Input**: HTTP GET request to `/api/example`
- **Output**: Response with status code 200 and containing the expected data.
```

**Flow:**

1. WebApplicationFactory
2. CreateClient()
3. Send HTTP requests
4. Assert responses

