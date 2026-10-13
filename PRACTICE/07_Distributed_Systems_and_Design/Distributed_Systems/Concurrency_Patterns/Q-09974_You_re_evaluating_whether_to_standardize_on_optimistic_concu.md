---
type: "practice"
difficulty: "Deep"
---

# You're evaluating whether to standardize on optimistic concurrency control across the org's data-access layer. At what write-contention profiles does OCC's retry cost become an architectural liability, and how do you give teams a data-driven rule for OCC vs pessimistic vs partitioned-single-writer?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Optimistic Concurrency Control (OCC) is a powerful tool to handle concurrent updates, but its effectiveness depends on write contention levels. At high write contention, OCC’s retry mechanism can become an architectural liability due to increased latency and resource usage. We need to provide teams with data-driven rules for choosing between OCC, pessimistic concurrency control, and partitioned single-writer models based on the specific write contention profiles.

#### Triaging the Question

The question is asking us to evaluate when optimistic concurrency control (OCC) becomes less advantageous compared to other concurrency management techniques. Specifically, we need to identify the thresholds at which OCC's retry mechanism introduces significant overhead. Additionally, the goal is to provide guidelines for developers to make informed decisions among OCC, pessimistic concurrency control, and partitioned single-writer models.

#### Assumptions
The system experiences varying levels of write contention.
We are focusing on a data-access layer that handles updates to shared resources.
The decision-making process involves analyzing both the technical feasibility and performance impact.
Teams need clear guidance to choose between different concurrency management strategies based on their workload characteristics.

#### First-Principles Solution
Write Contention Profiles
Low Write Contention:
In scenarios with low write contention, OCC typically performs well without frequent retries. The overhead is minimal, making it a good choice for many use cases.
Moderate Write Contention:
At moderate levels of write contention, the frequency of retries starts to increase, but the impact may still be manageable depending on the retry logic and backoff strategy used.
High Write Contention:
In highly concurrent environments with frequent writes, OCC's retry mechanism can become a bottleneck. The increased latency due to retries can degrade performance significantly.

#### Trade-Offs

OCC vs Pessimistic Concurrency Control (PCC):

#### Pros
of OCC:
Reduced locking overhead.
Can scale better in highly concurrent environments.

#### Cons
of OCC:
Increased latency due to retries.
Higher memory and CPU usage for retry logic.
PCC:
Reduces the need for frequent retries, leading to lower latency.
Easier to implement and reason about.

OCC vs Partitioned Single-Writer Model:

#### Pros
of OCC:
Better scalability due to no single point of contention.

#### Cons
of OCC:
Increased complexity in managing retries across partitions.
Partitioned Single-Writer:
Simplified concurrency management within each partition.
Reduced need for complex retry logic.
Rule of Thumb
For low write contention, stick with OCC as it minimizes locking and is easier to implement.
At moderate levels, monitor the frequency of retries. If they exceed a certain threshold (e.g., 5% of total writes), consider switching to PCC or partitioned single-writer models.
For high write contention, adopt either PCC or partitioned single-writer strategies to avoid the performance penalties associated with frequent retries.

#### Closing Summary

By understanding the write contention profiles and their impact on OCC’s retry mechanism, we can provide data-driven rules for choosing between OCC, pessimistic concurrency control, and partitioned single-writer models. This approach ensures that teams make informed decisions based on their specific workload characteristics, leading to more robust and efficient systems.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the performance impact of frequent retries in high write contention scenarios.

#### Follow-Up
How do we measure write contention? What tools can we use to monitor retry frequencies?

#### Example Scenario

#### Cons
ider a system where 95% of writes are low-contention operations, and only 5% involve moderate contention. In this case, OCC would be the optimal choice due to its reduced locking overhead. However, if more than 20% of writes experience high contention, we may need to switch strategies.

**Comparison:**

```
Contention Level	Frequency (%)	Recommended Strategy
Low	95	OCC
Moderate	4	Monitor Retries
High	1	PCC or Single-Writer
```

**Code:**

```python
# Example retry logic in Python for OCC
def update_data(data):
    retries = 3
    while retries > 0:
        try:
            # Attempt database update
            db.update(data)
            break
        except ConcurrencyException:
            retries -= 1
    if retries == 0:
        raise DataUpdateFailed()
```

**Example input / output:**

```text
Input:
- Write contention profile: moderate (20% of writes experience high contention)

Output:
- Decision: Switch to pessimistic concurrency control or partitioned single-writer model due to high retry frequency.
```

**Data:**

```json
{
  "writes_per_second": {
    "low_contention": 95,
    "moderate_contention": 4,
    "high_contention": 1
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Detect Write Contention
6. Is Contention High?
7. Switch to PessimisticConcurrency Control
8. Monitor Retry Frequency
9. Do Retries Exceed Threshold?
10. Switch to PartitionedSingle-Writer Model
11. Continue with OCC

