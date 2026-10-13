---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design tenant-aware and model-aware observability so you can answer 'is this one customer's inference degraded' or 'is this one model drifting' without scanning all logs across a shared multi-tenant serving platform.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tenant Isolation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Tenant_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of designing tenant-aware and model-aware observability in a multi-tenant cloud environment, we need to ensure that we can isolate and analyze performance metrics and drift indicators specific to individual tenants and models without having to scan all logs across the shared platform. The key idea is to leverage structured logging, metadata tagging, and a robust observability framework that supports fine-grained querying.

#### Triage

The question is essentially asking how to design an observability system that can distinguish between different tenants and their associated models in a multi-tenant setting. The interviewer wants to ensure that we can effectively monitor the performance and drift of specific tenant-model pairs without impacting the overall system's scalability or data privacy. We need to make several assumptions, such as the availability of metadata about tenants and models, and the use of structured logging.

#### First Principles Solution
Metadata Tagging: Each log entry should include metadata that uniquely identifies the tenant and model associated with it.
Structured Logging: Implement a standardized format for logs to ensure consistency and ease of parsing.
Tenant-Specific Observability Dashboards: Create dashboards tailored to each tenant, showing their specific metrics.
Model-Specific Monitoring: Set up monitoring rules that can be applied per model, allowing us to track drift or performance issues.
Step-by-Step Solution

Metadata Tagging:

Use a logging framework (e.g., Logstash) to tag each log entry with tenant and model identifiers.
For example, the log might look like this: {"tenant_id": "t001", "model_name": "mn002", ...}.

Structured Logging:

Define a standard schema for logs (e.g., JSON) to ensure consistency.
Use a centralized logging service (e.g., ELK Stack, Splunk) that can parse and store these structured logs efficiently.

Tenant-Specific Observability Dashboards:

Implement a dashboard system that allows creating views based on tenant ID.
These dashboards should provide metrics like inference latency, request rate, error rates, etc., specific to each tenant.

Model-Specific Monitoring:

Define monitoring rules (e.g., Prometheus queries) per model.
Use these rules to track drift or performance issues in real-time.
Implement alerting mechanisms for critical metrics that exceed predefined thresholds.

#### Closing Summary

By implementing metadata tagging and structured logging, we can ensure that logs are easily traceable to individual tenants and models. Tenant-specific dashboards and model-specific monitoring rules provide the necessary isolation needed to answer questions like "is this one customer's inference degraded" or "is this one model drifting." This approach ensures both scalability and privacy, as only relevant data is processed for each tenant-model pair.

#### Common Mistakes/Follow-Ups
Mistake: Not using metadata tagging can lead to logs being mixed up across tenants, making it difficult to isolate issues.

#### Follow-Up
How would you handle situations where a model is shared between tenants? Would the observability design need adjustments?

**Comparison:**

```
Tenant ID	Model Name	Avg Inference Latency (ms)
t001	mn002	48
```

**Code:**

```python
# Example of Python code for adding tenant and model metadata to logs
import json

def log_inference(tenant_id, model_name, inference_result):
    message = {
        "tenant_id": tenant_id,
        "model_name": model_name,
        "inference_result": inference_result
    }
    # Log the structured message using a logging framework like Python's built-in logging or an external service
    print(json.dumps(message))
```

**Example input / output:**

```

Example input log entry:

JSON
Copy
{
  "tenant_id": "t001",
  "model_name": "mn002",
  "inference_result": {
    "latency_ms": 50,
    "success": true
  }
}

Output for tenant-specific dashboard:

Tenant ID	Model Name	Avg Inference Latency (ms)
t001	mn002	48

Alert on model drift:

Rule: If the inference latency of mn002 exceeds 50 ms for more than 3 consecutive minutes, trigger an alert.
```

**Data:**

```json
{
  "tenant_id": "t001",
  "model_name": "mn002",
  "inference_result": {
    "latency_ms": 50,
    "success": true
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Log Entry
6. Is Metadata Tagged?
7. METADATA TAGGED
8. Parse and Store Log inCentralized Service
9. Tag and Resubmit Log
10. Tenant-Specific Dashboard
11. Tenant ID Match?
12. Detailed Metrics Available
13. Next Tenant

