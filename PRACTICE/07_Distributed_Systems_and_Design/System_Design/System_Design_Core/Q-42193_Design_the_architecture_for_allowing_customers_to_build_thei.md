---
type: "practice"
difficulty: "Expert"
---

# Design the architecture for allowing customers to build their own dashboards/reports over your data (embedded analytics) without letting a badly-written customer query take down your OLTP or leak another tenant's rows. Where does the query execution boundary live?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an architecture that allows customers to build their own dashboards/reports over your data while ensuring robustness and security, we need to focus on the query execution boundary and ensure that no single customer can impact the performance or security of other tenants. The key challenge is to isolate each tenant's queries from one another and prevent poorly-written queries from causing resource exhaustion.

This problem involves several layers: front-end, back-end, database, and possibly a caching layer for efficiency. Let’s break down the solution step by step:

Front-End Interface: This is where customers interact with your system to build dashboards/reports. The UI should be intuitive but also restrict certain types of complex queries that could cause performance issues.
Query Parsing and Validation: Before hitting the database, we need a robust query validation mechanism to ensure all queries are safe and efficient. This can include syntax checking, parameter validation, and schema enforcement.
Execution Boundary and Isolation: The execution boundary should be clearly defined between tenant data to prevent one customer from accessing another's data or causing resource starvation.
Query Execution Engine: A dedicated query engine should handle the actual execution of queries, ensuring that resources are allocated based on tenant-specific quotas.
Resource Management and Monitoring: Implementing a sophisticated resource management system is crucial for monitoring usage patterns and enforcing limits to prevent abuse.

Let's dive into these components in more detail:

#### Triage

Question Probing: The core of the question is about designing an architecture that allows flexible data exploration while ensuring robustness, security, and performance isolation. We need to consider how we can enforce query safety without overloading our systems or leaking data between tenants.

#### Assumptions

Each tenant has a unique set of data.
Tenants may write complex SQL queries for their reports/dashboards.
The system should handle multiple concurrent users efficiently.
There's a need to prevent poorly-written queries from causing performance issues.

#### First Principles Solution
Front-End Interface: The front-end application will provide a user-friendly interface where customers can create and save their dashboard configurations. It will also include client-side validation to ensure that the queries are not overly complex or resource-intensive before they reach the server.
Query Parsing and Validation: On the server side, we implement a robust query parser and validator that checks for SQL injection, syntax errors, and potential performance issues (e.g., using too many joins or selecting large datasets). This step should be done in real-time to prevent problematic queries from being executed.
Execution Boundary and Isolation: The execution boundary must ensure that each tenant's data is isolated from others. This can be achieved by partitioning the database schema, using row-level security (RLS), and implementing a query engine that enforces these boundaries during execution.
Query Execution Engine: A separate query execution engine should handle the actual processing of queries. It can use parallel execution to distribute workloads across multiple nodes if needed. The engine should also implement resource quotas per tenant to prevent one customer from consuming too many resources.
Resource Management and Monitoring: Implement a system that monitors resource usage in real-time. This includes CPU, memory, I/O, and network usage metrics for each tenant. If thresholds are exceeded, the system can automatically scale up/down or throttle queries.

#### Closing Summary

In summary, we need to architect an end-to-end solution that ensures security, performance, and robustness. The key components include a secure front-end interface, query validation, execution boundary isolation, dedicated query engines with resource quotas, and real-time monitoring for resource management.

#### Common Mistakes/Follow-Ups
Not Implementing Client-Side Validation: Assuming client-side checks are unnecessary can lead to malicious users bypassing server-side validation.
Ignoring Resource Isolation: Poorly isolating tenants' data can result in queries affecting one another, leading to performance degradation or even failures.
Insufficient Monitoring and Scaling Mechanisms: Without proper monitoring and scaling strategies, the system may become overwhelmed during peak usage periods.

To address these follow-ups:

Client-Side Validation: Include client-side validation as a first line of defense but ensure that all critical checks are also performed on the server side.
Resource Isolation: Use techniques like RLS and query partitioning to enforce strict isolation between tenants.
Monitoring and Scaling: Implement comprehensive monitoring tools and auto-scaling mechanisms to dynamically adjust resources based on tenant needs.

**Comparison:**

```
Tenant	Quota (CPU, Memory)	Reports
T1	200, 4GB	5
T2	300, 8GB	7
Resource Usage Monitoring Metrics
Tenant ID
----------
T1
T2
```

**Code:**

```python
# Example of a simple query validation function
def validate_query(query):
    try:
        # Parse the SQL query using a library like sqlparse
        parsed_query = sqlparse.parse(query)
        if len(parsed_query) > 1:
            return False, "Too many queries in one statement"

        # Check for large SELECT statements or complex joins
        for token in parsed_query[0].tokens:
            if isinstance(token, Identifier):
                table_name = token.value
                if is_large_table(table_name):  # Custom function to check table size
                    return False, "Query selects a large table"

        return True, "Valid query"
    except Exception as e:
        return False, str(e)
```

**Example input / output:**

```text
Input (Client):
- Query: "SELECT * FROM sales ORDER BY date DESC LIMIT 10"

Output (Server Validation):
- Validated as True
- No issues found

Input (Client):
- Query: "SELECT * FROM customers JOIN orders ON ... WHERE customer_id = '123' AND order_date > CURRENT_TIMESTAMP - INTERVAL 1 YEAR"

Output (Server Validation):
- Validated as False
- Reason: "Query selects a large table"
```

**Data:**

```yaml
# Example tenant data schema
tenant:
  id: string
  name: string
  quota: {
    cpu: int,
    memory: int,
    io: int,
    network: int
  }
  reports:
    - dashboard_id: string
      query: string
```

**Flow:**

1. Valid
2. Invalid
3. Within Limits
4. Exceeded Limits
5. User Input
6. Validate Query
7. Execute Query
8. Reject Query and Notify User
9. Check Quotas
10. Process Query
11. Scale Up Resources, ThenProcess Query

