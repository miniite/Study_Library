---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between reprocessing data and simply re-running a failed task?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Checkpointing and Recovery](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Checkpointing_and_Recovery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Reprocessing data involves running a full pipeline, often from source to destination, in order to ensure that all processed data matches the current state of the sources. Simply re-running a failed task means processing only the failed or changed subset of data, which can be more efficient and targeted.

To understand this difference, let's break down each concept step-by-step.

**Comparison:**

```
Stage	Reprocessing (Full Pipeline)	Rerunning Failed Task
Source Data Read	Yes, from start	No, only failed file
Transform Logic Execution	Yes, full transform	Only for the failed input
Destination Load	Yes, complete reload	Not necessary if data didn't change
```

**Code:**

```text
# Example pseudo-code for a simple data pipeline

def extract_data(source):
    # Logic to read data from source
    pass

def transform_data(raw_data):
    # Transform logic
    pass

def load_data(transformed_data, destination):
    # Load logic to the destination
    pass

pipeline = Pipeline(extract_data, transform_data, load_data)
```

**Example input / output:**

```text
Input:
- A task in the pipeline fails while transforming data.

Output:
- Reprocessing: The entire pipeline runs from the source to load the transformed data.
- Re-running failed task: Only the transformation step is rerun with the input data.
```

**Data:**

```json
{
  "source": "S3 bucket",
  "destination": "PostgreSQL database"
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. Start
5. Is task failure in sourceextraction?
6. Is task failure intransformation?
7. Run full pipeline
8. Extract new data only fromfailed source
9. Transform data with updatedrules
10. Load data into destination
11. End
12. Transform failed file only
13. Load data into destination

