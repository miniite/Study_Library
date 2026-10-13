---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer asks why you would not run your primary database on a single large instance 'to keep it simple'. What failure and maintenance realities push you toward a managed multi-AZ setup?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Separating Compute and Storage](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Separating_Compute_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let's quickly outline why running a primary database on a single large instance might seem appealing—simplicity and cost efficiency are often cited as key benefits. However, in practice, these advantages come with significant trade-offs that make a managed multi-AZ setup more robust and maintainable. This setup addresses critical concerns like availability, performance under load, disaster recovery, and ease of maintenance.

#### Triage
What the Question Is Really Asking

The interviewer is probing your understanding of cloud fundamentals, particularly around database resiliency and best practices. They want to know how you balance simplicity with robustness in a production environment. Key assumptions include familiarity with typical database failure modes and common maintenance tasks.

#### First-Principles Solution
Availability: Running a single large instance can introduce a single point of failure. If the instance fails, your application will experience downtime.
Performance Under Load: Single instances can struggle to scale vertically under heavy load without impacting performance.
Disaster Recovery and Data Protection: Relying on a single database increases the risk of data loss or corruption if the instance is compromised.
Maintenance: Managing a single large instance requires careful monitoring and tuning, which can be resource-intensive.

In contrast, a managed multi-AZ setup:

Provides high availability through automatic failover between AZs.
Distributes reads to multiple instances for better performance under load.
Ensures data replication across AZs for enhanced durability.
Simplifies maintenance by leveraging managed services that handle updates and scaling automatically.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while a single large instance might simplify the initial setup, it introduces significant risks in terms of availability, performance, and maintenance. A managed multi-AZ setup is more resilient and easier to maintain over time. This approach aligns with best practices for ensuring high availability and robustness in production environments.

#### Common mistakes
include underestimating the complexity of managing a single instance or ignoring the long-term costs associated with downtime and data loss. Follow-up questions might delve into specific use cases, cost considerations, or trade-offs between managed services and self-managed setups.

**Comparison:**

| AZ     | Instance ID          | Status       |
|--------|----------------------|--------------|
| us-west-1a  | i-0123456789abcdef0  | Active      |
| us-west-1b  | i-0fedcba9876543210  | Standby     |

**Code:**

```python
# Example code to demonstrate simple failover logic in multi-AZ setup
def ensure_high_availability(db_instance):
    try:
        db_instance.execute_query("SELECT * FROM important_data")
    except Exception as e:
        print(f"Primary instance failed: {e}")
        # Fallback to secondary instance or automatically managed AZ
```

**Example input / output:**

| Input (Query) | Output (Result) |
|---------------|----------------|
| `SELECT * FROM important_data` | [Data retrieved from us-west-1a] or [Automatically redirected to us-west-1b if primary fails] |

**Data:**

```json
{
  "db_instances": [
    {
      "instance_id": "i-0123456789abcdef0",
      "az": "us-west-1a"
    },
    {
      "instance_id": "i-0fedcba9876543210",
      "az": "us-west-1b"
    }
  ]
}
```

**Flow:**

1. Primary Instance OK
2. Failover Needed
3. User Query
4. Database Availability
5. Fetch Data
6. Redirect to Secondary AZ
7. Return Data
8. Automatically Failover toStandby

