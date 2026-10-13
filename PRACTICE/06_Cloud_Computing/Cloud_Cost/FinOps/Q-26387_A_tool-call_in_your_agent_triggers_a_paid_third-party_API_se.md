---
type: "practice"
difficulty: "Deep"
---

# A tool-call in your agent triggers a paid third-party API (search, enrichment, code sandbox) whose cost isn't on your cloud or LLM bill at all. How do you extend AI FinOps to cover this off-platform tool spend and attribute it to the agent task that caused it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shadow IT and Budget Evasion](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_IT_and_Budget_Evasion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To ensure comprehensive cost governance, we need to design a system that can accurately trace and attribute third-party API costs incurred by our AI agents back to the specific tasks or users who initiated them. This involves capturing metadata about each invocation, storing it, and then processing this data periodically to update financial reports.

#### Triaging the Question

The core challenge is to establish a robust mechanism for attributing the cost of external APIs (which are not billed directly by cloud providers) back to the individual AI tasks or users that triggered them. The goal is to provide visibility into total costs and ensure compliance with budgetary constraints. We need to consider how to log, store, and analyze this data effectively.

Step-by-Step Solution from First Principles
Metadata Capture: Each API invocation should be accompanied by metadata including:
Task ID
User ID (if applicable)
Timestamp
Request parameters
Duration of the API call
Log Aggregation and Storage:
Implement a logging system to capture this metadata.
Store logs in a centralized, searchable database like Elasticsearch or a data warehouse like BigQuery.
Cost Calculation Logic: Define rules for converting external API costs into our internal cost models (e.g., per-request pricing).
Reporting and Analytics:
Develop dashboards to visualize cost distribution by task, user, and time period.
Implement automated alerts for potential cost overruns.

#### Example
Workflow
When an AI agent triggers a third-party API call, metadata is captured and sent to the log storage system.
Periodically (e.g., daily), a batch job processes these logs to calculate costs based on predefined rules.
The calculated costs are then added to our internal financial reports for analysis.
Code Example
PYTHON
class ExternalAPIResponse:
    def __init__(self, request_id, user_id, timestamp, api_name, cost):
        self.request_id = request_id
        self.user_id = user_id
        self.timestamp = timestamp
        self.api_name = api_name
        self.cost = cost

def log_api_call(request_id, user_id, api_name, parameters=None):
    # Simulate API call and return a response with calculated cost
    cost = calculate_cost(api_name, parameters)
    response = ExternalAPIResponse(
        request_id=request_id,
        user_id=user_id,
        timestamp=datetime.now(),
        api_name=api_name,
        cost=cost
    )
    log_external_api_call(response)

def calculate_cost(api_name, parameters):
    # Example rule-based calculation; replace with actual logic
    if api_name == "Search":
        return 0.01 * len(parameters)
    elif api_name == "Enrichment":
        return 0.02 * (len(parameters) / 1000)

def log_external_api_call(response):
    # Log the response to a database or queue for later processing
    logging.info(f"API Call: {response}")

# Example usage
log_api_call("task-123", "user-456", "Search", parameters=["query1", "query2"])
Sample Data Schema
JSON
{
  "request_id": "string",
  "user_id": "string",
  "timestamp": "datetime",
  "api_name": "string",
  "cost": "float"
}

#### Common Mistakes
and Follow-Ups
Over-Engineering: Avoid making the system overly complex if basic metadata is sufficient.
Ignoring Edge Cases: Ensure to handle cases where no API call is made or there are unexpected parameters.

#### Performance Considerations
Batch processing of logs can be resource-intensive; consider optimizing this step.

#### Closing Summary

By implementing a systematic approach to capturing and analyzing metadata from external API calls, we can achieve comprehensive cost governance for our AI agents. This involves collecting detailed logs, applying cost calculation logic, and generating actionable reports. Proper design ensures that all costs are accurately attributed and managed within budget constraints.

**Comparison:**

```
Request ID	User ID	Timestamp	API Name	Cost
task-123	user-456	2023-10-09T14:48:00	Search	0.02
```

**Code:**

```python
class ExternalAPIResponse:
    def __init__(self, request_id, user_id, timestamp, api_name, cost):
        self.request_id = request_id
        self.user_id = user_id
        self.timestamp = timestamp
        self.api_name = api_name
        self.cost = cost

def log_api_call(request_id, user_id, api_name, parameters=None):
    # Simulate API call and return a response with calculated cost
    cost = calculate_cost(api_name, parameters)
    response = ExternalAPIResponse(
        request_id=request_id,
        user_id=user_id,
        timestamp=datetime.now(),
        api_name=api_name,
        cost=cost
    )
    log_external_api_call(response)

def calculate_cost(api_name, parameters):
    # Example rule-based calculation; replace with actual logic
    if api_name == "Search":
        return 0.01 * len(parameters)
    elif api_name == "Enrichment":
        return 0.02 * (len(parameters) / 1000)

def log_external_api_call(response):
    # Log the response to a database or queue for later processing
    logging.info(f"API Call: {response}")

# Example usage
log_api_call("task-123", "user-456", "Search", parameters=["query1", "query2"])
```

**Example input / output:**

```text
Input:
log_api_call("task-123", "user-456", "Search", parameters=["query1", "query2"])

Output:
{"request_id": "task-123", "user_id": "user-456", "timestamp": 2023-10-09T14:48:00, "api_name": "Search", "cost": 0.02}
```

**Data:**

```json
{
  "request_id": "string",
  "user_id": "string",
  "timestamp": "datetime",
  "api_name": "string",
  "cost": "float"
}
```

**Flow:**

1. Yes
2. No
3. API Call Initiated
4. Is API Triggered?
5. Log Metadata
6. Process Logs Periodically
7. Generate Reports
8. End

