---
type: "practice"
difficulty: "Deep"
---

# How do you architect quota, rate-limiting, and throttling policy for Cosmos DB (429 handling) so a bursty microservice can't starve the shared account — and who owns that policy org-wide?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cloud Database Architecture](<../../../../01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Cloud_Database_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect effective quota, rate-limiting, and throttling policies for Cosmos DB in a shared environment, we need to ensure that bursty microservices don't overwhelm the system and starve other critical services. This involves understanding the concepts of 429 (Too Many Requests) errors, implementing proper limits and quotas at both application and database levels, and establishing clear ownership and governance within the organization.

First, let's clarify what we're dealing with. A 429 error indicates that the system has been rate-limited due to excessive requests from a client. In a shared account setup, it’s crucial to prevent any one service from exhausting resources intended for others. The key is to balance performance and fairness across all services.

#### Triaging the Question

The question asks us to design policies for quota, rate-limiting, and throttling in Cosmos DB for bursty microservices. It also seeks to understand who owns these policies organization-wide. Here are our assumptions:

We're working with a shared database account where multiple microservices share access.
The goal is to prevent any single service from overloading the system without compromising overall performance.
Step-by-Step Solution

Understanding 429 Errors:

A 429 error occurs when the client exceeds the rate limit for a particular API endpoint. In Cosmos DB, this means hitting the request throughput limits set by the service.

Setting Up Quotas and Rate-Limits:

Define total available throughput (RU/s) across all services.
Allocate quotas to each microservice based on their expected usage patterns.
Implement rate-limiting at both the application level and the database API layer.

Implementing Throttling Policies:

Configure Cosmos DB to return 429 errors when a service exceeds its allocated throughput.
Use Azure Policy or custom scripting in your microservices to handle these errors gracefully, such as implementing backoff strategies, retries with exponential delays, or reattempting requests after a cooldown period.
Detailed Steps

Define Service Throughput:

SQL
-- Example of setting up throughput for services A and B
ALTER DATABASE [DatabaseName] SET THROUGHPUT = 5000;

Quota Allocation:

Break down the total available RU/s among all services based on their historical usage patterns.
For instance, if you have three services, allocate 40%, 30%, and 30% of the throughput accordingly.

Rate-Limiting at Application Level:

PYTHON
# Python example using Flask framework
from flask import Flask, request

app = Flask(__name__)

max_requests_per_minute = 1000  # Example limit
requests_in_minute = {}

@app.before_request
def limit_request():
    remote_addr = request.remote_addr
    if remote_addr in requests_in_minute:
        current_count = requests_in_minute[remote_addr]
        if current_count >= max_requests_per_minute:
            return "Too many requests", 429
        else:
            requests_in_minute[remote_addr] += 1
    else:
        requests_in_minute[remote_addr] = 1

@app.route('/api')
def api_route():
    # Your API logic here
    pass
Ownership and Governance

The ownership of these policies is critical for maintaining a healthy system. Typically, the responsibility would be shared among DevOps teams, database administrators (DBAs), and service owners.

DevOps Teams: Responsible for setting up monitoring and alerting to detect and respond to 429 errors.
DBAs: Manage quotas and throughput settings within Cosmos DB.
Service Owners: Ensure their services respect the rate limits and handle 429 errors gracefully.

#### Closing Summary

Properly architected quota, rate-limiting, and throttling policies are essential for managing a shared Cosmos DB account. By understanding and implementing these strategies, we can prevent bursty microservices from overwhelming the system while ensuring fair usage across all services.

#### Common Mistakes/Follow-Ups
Not Configuring Appropriate Quotas: Not allocating enough or too much throughput can lead to either underutilization or resource exhaustion.
Ignoring 429 Handling in Microservices: Failing to handle 429 errors gracefully can result in application crashes and poor user experience.

**Comparison:**

| Service | Quota (RU/s) |
|---------|-------------|
| A       | 2000        |
| B       | 3000        |
| C       | 1500        |

**Code:**

```python
# Python Flask example for rate-limiting
from flask import Flask, request

app = Flask(__name__)

max_requests_per_minute = 1000  # Example limit
requests_in_minute = {}

@app.before_request
def limit_request():
    remote_addr = request.remote_addr
    if remote_addr in requests_in_minute:
        current_count = requests_in_minute[remote_addr]
        if current_count >= max_requests_per_minute:
            return "Too many requests", 429
        else:
            requests_in_minute[remote_addr] += 1
    else:
        requests_in_minute[remote_addr] = 1

@app.route('/api')
def api_route():
    # Your API logic here
    pass
```

**Example input / output:**

```text
Input: GET /api/resource

Output (Normal):
{
  "data": "Resource data"
}

Output (429 Error Handling):
{
  "error": {
    "code": 429,
    "message": "Too Many Requests",
    "retry-after-seconds": 30
  }
}
```

**Data:**

```json
{
  "quota": {
    "serviceA": 2000,
    "serviceB": 3000,
    "serviceC": 1500
  },
  "throughput": 6500
}
```

**Flow:**

1. Exceeds Rate Limit
2. Within Limits
3. Service Requests
4. Rate Limit
5. Return 429
6. Resource
7. Backoff and Retry

