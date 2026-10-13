---
type: "practice"
difficulty: "Deep"
---

# Walk me through choosing between remote_write to a long-term store versus federation for feeding a central AIOps analytics cluster.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prometheus TSDB and Scrape Health](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to use remote_write or federation in a time-series database (like Prometheus or InfluxDB) to feed a central AIOps analytics cluster, we need to consider several factors including performance, consistency, fault tolerance, and ease of management. Remote_write involves writing data directly from remote sources to a long-term storage backend, whereas federation fetches and processes data remotely before forwarding it. Here’s how you can reason through the decision:

Firstly, let's understand what each approach entails:

Remote Write: Data is sent directly from collectors or scrape jobs to the long-term store, bypassing any intermediate processing.
Federation: Data is collected by a federation server and processed centrally before being forwarded.

#### Triage
What the Question Is Asking

The core of the question involves understanding whether it’s better to push data to a long-term storage backend in real-time or process and aggregate data centrally before storing. The interviewer might be probing your ability to weigh trade-offs, such as performance versus consistency, and considering how these choices impact the overall architecture.

#### First-Principles Solution
Performance
Remote Write: Direct writes can reduce latency but may introduce network load on the long-term storage backend.
Federation: Centralized processing could increase local latency but offloads some of the processing burden from remote nodes, potentially improving performance for downstream queries.

#### Cons
istency
Remote Write: Potential issues with data loss if the long-term store goes down.
Federation: Ensures that only processed and validated data reaches the analytics cluster, enhancing consistency and reducing noise.
Fault Tolerance
Remote Write: Simplifies fault tolerance as it's a direct write operation. If any single node fails, others can continue to operate independently.
Federation: Requires careful setup of redundancy in federated servers and long-term storage backends to ensure high availability.
Ease of Management
Remote Write: Easier to manage with simpler configurations but may require more powerful long-term storage systems.
Federation: More complex to set up and maintain due to additional components like federation servers, but can simplify long-term storage by reducing data volume.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the choice between remote_write and federation depends on your specific requirements. If performance is critical and you have a robust network infrastructure, remote_write might be preferable. However, if consistency and validation of data are paramount, especially in complex environments with many remote collectors, federated processing can provide better control.

A common mistake is overlooking the impact of additional latency introduced by federation servers, which can affect real-time analytics use cases. Always consider the trade-offs between performance and reliability when making this decision.

Common Follow-Ups
What if we have a hybrid approach?
How do we handle different data volumes for remote_write vs. federation?
Can you walk me through an example scenario where one would be better than the other?

**Comparison:**

```
Factor	Remote Write	Federation
Performance	Direct writes	Central processing
Consistency	Risk of data loss	Processed & validated
Fault Tolerance	Independent nodes	Redundancy required
Management	Simpler	More complex
```

**Code:**

```python
# Example of setting up remote_write in Prometheus configuration
remote_write:
  - url: http://long_term_store:8086/write
    basic_auth_user: user
    basic_auth_password: pass

# Example federation server setup in InfluxDB
[http]
  enabled = true
  bind-address = ":8085"
  external-url = "https://influxdb.example.com/"
  api-enabled = true
  allow-anonymous = false
  auth-enabled = true
  [federation]
    enabled = true
```

**Example input / output:**

```text
Remote Write Input -> Output:
Collector → Long-Term Store (Directly)

Federation Input -> Output:
Collector → Federation Server → Long-Term Store (Processed)
```

**Data:**

```json
{
  "remote_write": [
    {
      "url": "http://long_term_store:8086/write",
      "basic_auth_user": "user",
      "basic_auth_password": "pass"
    }
  ],
  "federation": {
    "enabled": true,
    "external_url": "https://influxdb.example.com/",
    "api_enabled": true,
    "allow_anonymous": false,
    "auth_enabled": true
  }
}
```

**Flow:**

1. Collector
2. (Direct Write)
3. Long-Term Store
4. Federation Server
5. Processed Data
6. Long-Term Store

