---
type: "practice"
difficulty: "Intermediate"
---

# You batch-score 2 billion rows nightly for a model and it must finish in a 4-hour window but one partition is 50x larger than the rest. Explain data skew in this context and three ways to fix it, plus how to make the job idempotent for safe resume.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In a distributed system for batch-scoring 2 billion rows nightly, one partition being 50x larger than the others can lead to significant delays and inefficiencies. This scenario highlights data skew, which we need to address to ensure the job completes within the 4-hour window. Three effective strategies are: dynamic workload rebalancing, partitioning by similar characteristics, and using idempotent jobs for safe resumption.

#### Triaging the Question

The question is probing your understanding of distributed systems, specifically how to handle data skew in large-scale batch processing tasks. The key assumptions here include a requirement to complete processing within 4 hours and the presence of an unusually large partition that could be causing delays. We need to explore ways to balance the workload more evenly across nodes and ensure that the job can safely resume if it gets interrupted.

#### First-Principles Solution
1. Dynamic Workload Rebalancing

One approach is to dynamically rebalance the workload during runtime. This involves monitoring each partition's progress and redistributing tasks from larger partitions to smaller ones as needed. To implement this, you can use an orchestrator that periodically checks the status of each partition and triggers a reassignment when necessary.

2. Partitioning by Similar Characteristics

A more proactive approach is to partition data based on characteristics that ensure even distribution among nodes. For instance, if your model input has some natural clustering (e.g., user IDs), you can create partitions that contain similar-sized subsets of the data. This reduces the likelihood of any single partition being significantly larger than others.

3. Idempotent Jobs for Safe Resumption

To ensure safety and reliability in case of interruptions, making the job idempotent is crucial. An idempotent job can be safely restarted multiple times without changing its outcome. This involves maintaining a state tracking mechanism (e.g., using a database or distributed ledger) to keep track of which rows have been processed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Dynamic workload rebalancing and partitioning by characteristics are effective strategies for handling data skew, but they require careful implementation to avoid overcomplicating the system. Idempotent jobs ensure that processing can be safely resumed even if a node fails or is interrupted mid-processing.

#### Common mistakes
might include not fully understanding the data distribution and thus failing to rebalance effectively, or implementing idempotency in a way that introduces unnecessary complexity (e.g., overusing locks or distributed transactions).

Trialing Further

Code Example:

PYTHON
from collections import defaultdict

class DataProcessor:
    def __init__(self):
        self.processed_rows = defaultdict(int)

    def process_row(self, row_id):
        if not self.processed_rows[row_id]:
            # Process the row (simulated here)
            print(f"Processing row {row_id}")
            self.processed_rows[row_id] += 1

processor = DataProcessor()
for i in range(20):  # Simulating 2 billion rows
    processor.process_row(i)

Dry-Run Input->Output:

PLAINTEXT
Processing row 0
Processing row 1
...
Processing row 19

Common Follow-Ups:

How do you ensure the reassignment of tasks is efficient?
Can you discuss potential trade-offs between these strategies and other factors like resource utilization?

**Comparison:**

| Row ID | Processed |
|--------|-----------|
| 0      | Yes       |
| 1      | Yes       |
| ...    |           |
| 19     | Yes       |

**Code:**

```python
from collections import defaultdict

class DataProcessor:
    def __init__(self):
        self.processed_rows = defaultdict(int)

    def process_row(self, row_id):
        if not self.processed_rows[row_id]:
            print(f"Processing row {row_id}")
            self.processed_rows[row_id] += 1
```

**Example input / output:**

```text
Processing row 0
Processing row 1
...
Processing row 19
```

**Data:**

```json
{
  "row_ids": [0, 1, 2, ..., 19]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is row processed?
5. Mark as processed andprocess
6. Next row
7. End

