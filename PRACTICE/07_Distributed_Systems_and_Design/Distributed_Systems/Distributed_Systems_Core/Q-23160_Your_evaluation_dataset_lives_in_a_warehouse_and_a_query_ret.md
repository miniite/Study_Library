---
type: "practice"
difficulty: "Intermediate"
---

# Your evaluation dataset lives in a warehouse and a query returns slightly different rows run to run, making eval scores non-reproducible. Explain how snapshot isolation or dataset versioning like Iceberg time-travel fixes this and why immutable eval sets matter.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating models in a distributed system, consistency across runs is crucial to ensure reproducibility and reliability of scores. In your scenario, the evaluation dataset lives in a warehouse where queries return different rows each time they are run, making eval scores non-reproducible. Snapshot isolation or using immutable datasets like Iceberg can fix this issue by ensuring that a consistent view of the data is provided across multiple executions.

Snapshot isolation works by allowing transactions to read a version of the database as it was at the start of their execution, rather than the current state. This means that each query sees a snapshot of the dataset from a specific point in time, which ensures consistency and reproducibility.

Let's break down how this works:

Understanding Snapshot Isolation:

Each transaction starts by reading the database at the same consistent point-in-time.
Data is not updated until all transactions are completed, ensuring that no partial updates are visible to other transactions.
This approach helps in maintaining consistency across multiple runs of a query.

Why Immutable Eval Sets Matter:

#### Cons
istency Across Runs: By using an immutable dataset, each evaluation run uses the same consistent snapshot of data, which ensures that results are reproducible and comparable.
Avoiding Data Variability: In mutable datasets, updates can occur between runs, causing discrepancies in scores. Snapshot isolation or versioned datasets prevent such changes from affecting the evaluation.

How Iceberg Dataset Versioning Works:

Versioned Snapshots: Iceberg maintains a history of all data changes over time.
Time Travel Query: You can query specific versions of the dataset, effectively "traveling back" to any point in the past for consistent evaluations.
Guaranteed Consistency: By leveraging these features, you ensure that each evaluation run uses the exact same version of the data.

#### Implementation
Steps:

Define a Stable Baseline: Identify and document the baseline snapshot or version of the dataset to be used in all evaluation runs.
Automate Snapshot Selection: Ensure that your evaluation code selects the correct snapshot from Iceberg or configures the database for snapshot isolation.
Monitor and Validate: Regularly monitor data changes and validate that snapshots are correctly applied to maintain reproducibility.

**Code:**

```python
# Example Python code snippet to demonstrate how to use a specific version of an Iceberg table

from pyiceberg.expressions import And, EqualTo, GreaterThanOrEqual, LessThan
from pyiceberg.io import read_table

def get_evaluation_snapshot(table_path, version):
    # Load the Iceberg table
    table = read_table(table_path)

    # Define a predicate to select specific versions (e.g., based on time or change ID)
    snapshot_predicate = GreaterThanOrEqual(table.change_id(), version)

    # Read the dataset as of the specified version
    return table.new_scan().where(snapshot_predicate).execute()

# Example usage
snapshot_version = "2023-10-01"
evaluation_snapshot = get_evaluation_snapshot("/path/to/your/table", snapshot_version)
```

**Example input / output:**

```text
Request to evaluate model on version 2023-10-01:
- Input table path: /path/to/your/table
- Desired snapshot version: 2023-10-01

Output (evaluation scores):
- Precision: 0.85
- Recall: 0.92
- F1 Score: 0.88
```

**Flow:**

1. Yes
2. No
3. Initiate Evaluation
4. Use Snapshot Isolation?
5. Select Consistent DatasetSnapshot
6. Update or Modify Data inPlace
7. Evaluate Model with SelectedSnapshot
8. Run Evaluation on Latest Data
9. Report Results (Consistent)
10. Report Results (PotentiallyInconsistent)

