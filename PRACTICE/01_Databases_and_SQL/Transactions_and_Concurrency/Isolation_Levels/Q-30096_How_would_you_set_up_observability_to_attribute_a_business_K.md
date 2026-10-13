---
type: "practice"
difficulty: "Deep"
---

# How would you set up observability to attribute a business KPI regression (e.g. oversells) back to an isolation-level anomaly rather than a code bug across a large service graph?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ From Serializable to Sagas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/From_Serializable_to_Sagas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of attributing a business KPI regression, such as overselling, back to an isolation-level anomaly in a large service graph, we need to establish robust observability practices that can trace transactions across various services and pinpoint which isolation level might be causing the issue. This requires setting up comprehensive logging, monitoring, and alerting mechanisms.

#### Triage

The key question here is how to set up observability so that we can accurately identify if a specific transactional anomaly (e.g., an oversell) is due to a misconfigured isolation level rather than a bug in the application code. The goal is to detect when transactions are being mishandled and to trace these back to the relevant isolation setting.

#### First-Principles Solution
Transaction Logging: Implement detailed logging for each transaction, including key metadata such as the operation type, involved services, timestamps, and isolation level.
Monitoring & Alerting: Set up real-time monitoring and alerting based on predefined KPIs like oversells to trigger investigation when deviations are detected.
Correlation ID Management: Use a unique correlation ID for each transaction that spans multiple services. This helps in correlating events across different components.
Root Cause Analysis Tools: Employ APM tools to trace the flow of transactions and pinpoint where anomalies occur.
Step-by-Step Solution

Transaction Logging:

Log every transaction with detailed metadata: transaction_id, service_name, operation_type, start_time, end_time, isolation_level, and any relevant business KPIs like oversold_units.

#### Example
log entry:
JSON
{
  "transaction_id": "tx12345",
  "service_name": "InventoryService",
  "operation_type": "UPDATE_STOCK",
  "start_time": "2023-10-01T12:00:00Z",
  "end_time": "2023-10-01T12:05:00Z",
  "isolation_level": "RR",
  "oversold_units": 2
}

Monitoring & Alerting:

Configure real-time monitoring to detect anomalies in KPIs like oversolds.
Set up alerts for thresholds that indicate potential issues, such as a sudden increase in oversold_units.

#### Example
alert rule: If the average oversold_units over the last 5 minutes exceeds 10 units, trigger an alert.

Correlation ID Management:

Generate a unique correlation ID for each transaction that persists across services.
Pass this ID through all service calls to maintain context and traceability.

#### Example
of passing a correlation ID in HTTP headers: X-Request-ID: tx12345.

Root Cause Analysis Tools:

Use APM tools like Datadog, New Relic, or custom-built solutions to trace the flow of transactions.
Implement transaction tracing and distributed tracing to capture how data flows between services.

#### Example
setup in a microservices architecture using OpenTelemetry:
User Request
InventoryService
OrderService
PaymentService
ShippingService
Database
Database

#### Closing Summary

By implementing robust observability practices, we can effectively trace transactions and identify if a KPI regression like overselling is due to an isolation-level anomaly. This involves detailed logging, real-time monitoring, correlation ID management, and root cause analysis tools.

#### Common Mistakes/Follow-Ups
Mistake: Failing to log critical metadata such as isolation_level in transaction logs.

#### Follow-Up
Ensure that all services are instrumented correctly and that the logs capture necessary details for effective tracing.

**Code:**

```
MERMAID
Copy
graph TD;
  A[User Request] --> B[InventoryService];
  B --> C[OrderService];
  C --> D[PaymentService];
  D --> E[ShippingService];
  C --> F[Database];
  F --> G[Database];
  style A fill:#32CD32,stroke:#000,stroke-width:4px;
  style F fill:#FF4500,stroke:#000,stroke-width:4px;
```

**Example input / output:**

```text
Input:
Transaction with `isolation_level` set to 'RR' in a microservices environment.

Output:
Real-time monitoring alert triggered due to high `oversold_units`.

Root Cause Analysis:
- Logs show multiple transactions where `isolation_level` was incorrectly set.
- APM traces reveal that the issue stems from one specific service (`OrderService`) under heavy load.
```

**Data:**

```json
{
  "transaction_id": "tx12345",
  "service_name": "InventoryService",
  "operation_type": "UPDATE_STOCK",
  "start_time": "2023-10-01T12:00:00Z",
  "end_time": "2023-10-01T12:05:00Z",
  "isolation_level": "RR",
  "oversold_units": 2
}
```

