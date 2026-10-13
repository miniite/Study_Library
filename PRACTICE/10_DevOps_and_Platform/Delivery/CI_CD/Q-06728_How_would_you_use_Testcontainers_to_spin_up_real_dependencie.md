---
type: "practice"
difficulty: "Deep"
---

# How would you use Testcontainers to spin up real dependencies in CI?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Integration Testing and Test Doubles](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we will explore how to leverage Testcontainers to spin up real database instances during Continuous Integration (CI). The core idea is that using Testcontainers allows us to run a lightweight, throwaway instance of a database (like PostgreSQL or MySQL) in memory for each CI job. This approach ensures that our tests are isolated and can rely on consistent database state without the need for complex setup procedures.

#### Triage

The question asks about using Testcontainers to spin up real dependencies during CI. The key aspects being probed include:

Understanding of Testcontainers: How it works, its benefits over traditional test databases.
Integration with CI pipelines: How to configure and run Testcontainers within a CI environment.
Handling database isolation and consistency: Ensuring tests are isolated from each other while maintaining consistent data states.

#### First-Principles Solution
Setting Up Testcontainers: Configure Testcontainers in your build system (e.g., Maven, Gradle) to automatically start a lightweight instance of the chosen database.
Database Configuration: Update your application’s configuration to use the database URL provided by Testcontainers.
Test Initialization and Teardown: Write tests that initialize the database connection, run test cases, and close the connection after each run.
Detailed Steps
Add Dependencies:
For Maven, include Testcontainers dependencies in your pom.xml:
XML
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.17.6</version>
    <scope>test</scope>
</dependency>
Database Configuration:
Update your application’s configuration (e.g., application.yml or application.properties) to support database URL injection.
Initialize Testcontainers in Tests:
Create a test class using JUnit and use the @BeforeAll annotation to start the container before all tests.
JAVA
@SpringBootTest
@ContextConfiguration(initializers = DatabaseConfigInitializer.class)
public class MyServiceTest {
    private static PostgreSQLContainer<?> postgres;

    static {
        postgres = new PostgreSQLContainer<>("postgres:latest");
        postgres.start();
    }

    @Autowired
    private MyService myService;

    @BeforeEach
    void setUp() {
        // Setup data for tests if needed.
    }

    @AfterEach
    void tearDown() {
        // Clean up after each test.
    }
}
Use Database URL in Application Configuration:
Modify your application configuration to use the database URL provided by Testcontainers.
YAML
spring.datasource.url=${TESTCONTAINERS_POSTGRESQL_JDBC_URL}

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Not properly starting/stopping the container within test lifecycle methods, leading to resource leaks or failures in subsequent tests.
Failing to configure environment variables correctly for database URLs.
Ignoring test data setup/cleanup, which can lead to inconsistent states between tests.
Depth and Considerations
Performance: Testcontainers start quickly but might not be ideal for very performance-critical scenarios. Consider the overhead of starting a new instance for each CI job.
Scalability: For large-scale CI pipelines, ensure that test containers are well-managed to avoid resource contention or delays.
Final Thoughts

Testcontainers provide a powerful way to manage database dependencies in CI without setting up and tearing down actual databases. This approach helps maintain test isolation while ensuring consistent environments for each run.

**Code:**

```xml
<!-- Maven dependencies -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.17.6</version>
    <scope>test</scope>
</dependency>
```

**Data:**

```yaml
# application.yml
spring.datasource.url=${TESTCONTAINERS_POSTGRESQL_JDBC_URL}
```

