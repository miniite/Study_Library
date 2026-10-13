---
type: "practice"
difficulty: "Intermediate"
---

# What does the run_results.json artifact contain after a dbt run?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Fundamentals](<../../../../TOOLS/dbt/dbt_Core/dbt_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a dbt (data build tool) project, run_results.json is an important artifact that provides detailed information about the outcomes of each model and macro during a dbt run. This JSON file serves as a comprehensive log to help understand which models were executed successfully, failed, or encountered warnings.

#### Triage

The question asks about the contents of run_results.json. The key points we need to address are:

What information is stored in this file?
How does this differ from other dbt artifacts like target or compiled?

We assume that a dbt run has completed, and you have access to the run_results.json file.

#### First-Principles Solution

run_results.json is structured as an array of JSON objects, each representing the result of a single model or macro execution. Each object in this array contains information such as:

Model name: The unique identifier for the model.
Status: Whether the run was successful (success), failed (error), or had warnings (warn).
Node type: Indicates whether it’s a model, test, or macro.
Message: A detailed message explaining the outcome (e.g., error details, warning messages).
Timing information: Details on how long the execution took.

Let's break down what this means step-by-step:

Model Name and Node Type:

Each entry will have a name field that uniquely identifies the model.
The node_type field specifies whether it’s a model, test, or macro.

Status:

If status is "success", the model ran without any issues.
If status is "error", there was an error during execution.
If status is "warn", some warnings were issued but no critical errors occurred.

Message and Timing:

The message field provides more context about what happened, such as specific SQL errors or warnings.
Timing information (timing) gives the duration of the model run in milliseconds.

#### Example
Content

Here’s a simplified example to illustrate:

JSON
[
  {
    "name": "my_model",
    "node_type": "model",
    "status": "success",
    "message": "",
    "timing": [12345, 67890]
  },
  {
    "name": "test_my_model",
    "node_type": "test",
    "status": "warn",
    "message": "Column 'age' is not a valid integer type.",
    "timing": [34567, 78912]
  }
]

#### Closing Summary
and Common Mistakes/Follow-Ups

run_results.json provides critical insights into the dbt run’s success or failure. It helps in pinpointing issues quickly and is a valuable tool for troubleshooting. Common mistakes often involve not checking this file when issues arise, leading to wasted time searching through other artifacts.

For follow-ups, you might be asked about handling large datasets where run_results.json becomes cumbersome. In such cases, using dbt’s logging capabilities or integrating with monitoring tools like Grafana can provide real-time insights and alerting mechanisms.

**Code:**

```json
[
  {
    "name": "my_model",
    "node_type": "model",
    "status": "success",
    "message": "",
    "timing": [12345, 67890]
  },
  {
    "name": "test_my_model",
    "node_type": "test",
    "status": "warn",
    "message": "Column 'age' is not a valid integer type.",
    "timing": [34567, 78912]
  }
]
```

