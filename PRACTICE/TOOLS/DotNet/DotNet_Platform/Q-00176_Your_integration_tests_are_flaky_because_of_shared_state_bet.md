---
type: "practice"
difficulty: "Deep"
---

# Your integration tests are flaky because of shared state between the test host and a real DB — how would you structure WebApplicationFactory, test containers, and data isolation for reliable parallel tests?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our integration tests remain reliable in a .NET environment with shared state issues due to interactions with a real database, we need to isolate the test data effectively. This involves structuring WebApplicationFactory and leveraging test containers like SQL Server Docker containers for data isolation. We'll also explore setting up a clean-up strategy to avoid overlapping states between parallel tests.

#### Triaging the Question

The core of this question revolves around ensuring that our integration tests are not affected by shared state issues, particularly those arising from interactions with the real database. The key is to create isolated test environments where each test runs in its own context without interfering with others. This involves understanding and configuring WebApplicationFactory, utilizing test containers, and implementing a robust cleanup mechanism.

#### Assumptions
We are using .NET Core or .NET 5+.
The database we interact with is SQL Server.
We have control over the database setup and can use Docker containers for isolation.
Step-by-Step Solution
WebApplicationFactory Configuration

WebApplicationFactory is a powerful tool provided by ASP.NET Core that allows us to set up an in-memory application. By using this factory, we ensure that each test runs with a fresh instance of the application, which can be configured and mocked as needed.

CSHARP
public class CustomWebApplicationFactory : WebApplicationFactory<Startup>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // Additional configuration if required.
    }
}
Test Containers (SQL Server Docker Containers)

We will use SQL Server Docker containers to run a fresh instance of the database for each test. This ensures that no state is carried over from one test to another.

Dockerfile: Create a Dockerfile in our project to set up the environment.
Test Fixture Setup: In our test class, we will start and stop the Docker container before and after running tests.
CSHARP
public class IntegrationTest : IDisposable
{
    private readonly string _connectionString;

    public IntegrationTest()
    {
        // Start the SQL Server Docker container with a new unique port.
        var dockerContainer = new DockerSqlServer("1433", "myappdb");
        _connectionString = $"Server={dockerContainer.Host}:{dockerContainer.Port};Database=myappdb;User Id=sa;Password=mypassword;";
    }

    [Fact]
    public void Test()
    {
        // Use the database connection string here.
        using var context = new MyDbContext(_connectionString);
        // Perform test actions.
    }

    public void Dispose()
    {
        // Stop and clean up the Docker container.
        var dockerContainer = new DockerSqlServer("1433", "myappdb");
        dockerContainer.Stop();
    }
}
Data Isolation

To ensure that each test runs in an isolated environment, we need to manage the database state carefully. This can be achieved by using transactions or ensuring that the test setup and teardown methods are correctly implemented.

Transaction Scope: Use a using block with TransactionScope to wrap your database operations.
Test Setup and Teardown: Implement methods in your test fixture to set up the initial state and clean up after each test run.
CSHARP
public class IntegrationTest : IDisposable
{
    private readonly string _connectionString;

    public IntegrationTest()
    {
        // Start the SQL Server Docker container with a new unique port.
        var dockerContainer = new DockerSqlServer("1433", "myappdb");
        _connectionString = $"Server={dockerContainer.Host}:{dockerContainer.Port};Database=myappdb;User Id=sa;Password=mypassword;";
    }

    [Fact]
    public void Test()
    {
        using var transactionScope = new TransactionScope();
        try
        {
            // Use the database connection string here.
            using var context = new MyDbContext(_connectionString);
            // Perform test actions.
            Assert.True(/* condition */);
            transactionScope.Complete();
        }
        catch (Exception)
        {
            transactionScope.Dispose(); // Ensure rollback in case of error.
        }
    }

    public void Dispose()
    {
        // Stop and clean up the Docker container.
        var dockerContainer = new DockerSqlServer("1433", "myappdb");
        dockerContainer.Stop();
    }
}

#### Closing Summary

By using WebApplicationFactory to set up a fresh in-memory application, leveraging SQL Server Docker containers for data isolation, and managing transactions properly, we can ensure that our integration tests are reliable and do not suffer from shared state issues. This approach provides a robust environment for testing while maintaining high parallelism.

#### Common Mistakes/Follow-Ups
Not Starting/Stopping Containers Properly: Ensure you handle the lifecycle of Docker containers correctly to avoid resource leaks.
Leaky Transactions: Make sure transactions are properly committed or rolled back in every test case.
Race Conditions: Be cautious of race conditions that might arise if multiple tests are accessing and modifying the same data simultaneously.

**Comparison:**

| Test Method | Input Actions | Output Assertions |
|-------------|---------------|------------------|
| Test        | Execute SQL commands | Pass/Failed assertions |

**Code:**

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Startup>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // Additional configuration if required.
    }
}

public class IntegrationTest : IDisposable
{
    private readonly string _connectionString;

    public IntegrationTest()
    {
        // Start the SQL Server Docker container with a new unique port.
        var dockerContainer = new DockerSqlServer("1433", "myappdb");
        _connectionString = $"Server={dockerContainer.Host}:{dockerContainer.Port};Database=myappdb;User Id=sa;Password=mypassword;";
    }

    [Fact]
    public void Test()
    {
        using var transactionScope = new TransactionScope();
        try
        {
            // Use the database connection string here.
            using var context = new MyDbContext(_connectionString);
            // Perform test actions.
            Assert.True(/* condition */);
            transactionScope.Complete();
        }
        catch (Exception)
        {
            transactionScope.Dispose(); // Ensure rollback in case of error.
        }
    }

    public void Dispose()
    {
        // Stop and clean up the Docker container.
        var dockerContainer = new DockerSqlServer("1433", "myappdb");
        dockerContainer.Stop();
    }
}
```

**Example input / output:**

```text
Input:
- Test method [Test] in IntegrationTest class

Output:
- Database operations executed within a transaction scope.
- Assertions passed or failed based on test conditions.
- Docker container started and stopped as part of the test lifecycle.
```

**Data:**

```json
{
  "docker_container": {
    "host": "localhost",
    "port": "1433",
    "name": "myappdb"
  },
  "connection_string": "Server=localhost:1433;Database=myappdb;User Id=sa;Password=mypassword;"
}
```

**Flow:**

1. Yes
2. No
3. Start Docker Container
4. Set Up Database Context
5. Is Transaction Completed?
6. Rollback Transaction
7. Perform Test Actions
8. Assert Conditions
9. Commit Transaction
10. Stop Docker Container

