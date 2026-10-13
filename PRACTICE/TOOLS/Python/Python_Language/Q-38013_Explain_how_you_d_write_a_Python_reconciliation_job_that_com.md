---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd write a Python reconciliation job that compares source and target row counts and checksums per partition, and how you'd decide what counts as meaningful drift versus noise.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data Reconciliation Scripts](<../../../../TOOLS/Python/Python_Language/Data_Reconciliation_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a Python reconciliation job that compares source and target row counts and checksums per partition, we need to break down the problem into key components: reading data from sources and targets, calculating row counts and checksums, comparing them across partitions, and deciding on thresholds for meaningful drift. We'll start with the core logic and then discuss how to handle edge cases and define what constitutes noise versus meaningful drift.

First, let's lay out the steps:

Read Data: Extract data from both source and target databases.
Partition Data: Split the data into logical partitions (e.g., by date or region).
Calculate Metrics: Compute row counts and checksums for each partition.
Compare Metrics: Compare the metrics between source and target to identify drift.
Thresholding: Define thresholds to determine what constitutes meaningful drift.

**Comparison:**

```
Partition	Source Count	Target Count
2023-09	10,000	10,050
Partition	Source Checksums	Target Checksums
2023-09	0x87654321	0x87654321
```

**Code:**

```python
import pandas as pd
from sqlalchemy import create_engine

# Database connection strings
source_db_url = "postgresql://user:pass@localhost/source"
target_db_url = "postgresql://user:pass@localhost/target"

def read_data(db_url, partition_column):
    """Read data from a database and return a DataFrame."""
    engine = create_engine(db_url)
    query = f"SELECT * FROM table WHERE {partition_column} IS NOT NULL"
    return pd.read_sql(query, engine)

def calculate_metrics(df, partition_column):
    """Calculate row count and checksum for each partition."""
    df['checksum'] = df.apply(lambda row: hash(row.to_string()), axis=1)
    metrics = df.groupby(partition_column).agg({'id': 'count', 'checksum': lambda x: ','.join(x)}).reset_index()
    return metrics
```

**Example input / output:**

```text
# Example input data for a partition (date)
source_df = read_data(source_db_url, 'date')
target_df = read_data(target_db_url, 'date')

# Calculate metrics
source_metrics = calculate_metrics(source_df, 'date')
target_metrics = calculate_metrics(target_df, 'date')
```

**Data:**

```yaml
partitions:
  - date: "2023-09"
  - region: "North America"
```

**Flow:**

1. Yes
2. No
3. Yes
4. Noise
5. Drift
6. No
7. Yes
8. Read Data
9. Is Partition Column NULL?
10. Fetch Data Using SQL
11. Use DataFrame to Read Data
12. Calculate Metrics
13. Group By Partition &Aggregate
14. Compare Metrics
15. Compare Checksums
16. Check Row Count Drift
17. Noise or Drift?
18. Log as Normal Variation
19. Alert and Investigate
20. Check Checksum Differences
21. Checksums Match, Normal
22. Checksum Mismatch, FurtherInvestigation Needed

