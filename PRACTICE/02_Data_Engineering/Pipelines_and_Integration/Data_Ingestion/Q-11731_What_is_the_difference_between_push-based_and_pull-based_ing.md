---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between push-based and pull-based ingestion?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC vs Full Reload](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/CDC_vs_Full_Reload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data ingestion, understanding the difference between push-based and pull-based methods is crucial for designing effective pipelines. Push-based ingestion involves a source system actively sending data to a destination, while pull-based ingestion requires the destination to fetch data from the source at regular intervals. This answer will explore both methods, their characteristics, and when each might be preferable.

**Comparison:**

```
Method	Source Active?	Polling Frequency	Example Use Case
Push	Yes	Not Needed	Real-time trading data
Pull	No	Fixed or Variable	Historical sales data
```

**Code:**

```python
# Example of pull-based script using a simple polling mechanism (hypothetical)
import time

def pull_data_from_source():
    # Code to fetch data from the source system
    pass

while True:
    pull_data_from_source()
    time.sleep(60)  # Poll every minute
```

**Example input / output:**

```text
Input:
- A database with sales data that needs to be moved to a data warehouse.

Output:
- Data from the source is periodically copied to the destination.
```

**Data:**

```json
{
  "source": {
    "type": "Database",
    "connectionInfo": {"host": "localhost", "port": 3306, "username": "root"}
  },
  "destination": {
    "type": "Data Warehouse",
    "connectionInfo": {"host": "aws-dwh", "port": 5439, "dbname": "sales"}
  }
}
```

**Flow:**

1. Source
2. Push-Based Ingestion
3. Destination
4. Pull-Based Ingestion
5. Real-time Data
6. Historical Data

