---
type: "practice"
difficulty: "Expert"
---

# What's your position on watermarking as a first-class org-wide contract? How do you stop each team from inventing their own lateness semantics and creating a governance nightmare?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Watermarking is an essential tool in ensuring consistency and reliability across both batch and stream processing pipelines, making it a first-class contract in any organization. By establishing watermarking as a standardized practice, we can prevent each team from inventing their own lateness semantics, which would otherwise lead to significant governance issues and potential data integrity problems.

#### Triage
Understanding the Question

The question is essentially asking about how to enforce a unified approach to handling late arriving data across different teams within an organization. This involves ensuring that everyone follows the same conventions for dealing with late records, such as marking when a record is considered "late" based on some time watermark. The core issue here is maintaining consistency and avoiding the chaos that can arise from diverse or conflicting practices.

Tackling the Problem From First Principles

Definition of Watermarking: Watermarking involves setting a timestamp (watermark) to indicate when records are considered valid up to a certain point in time. This watermark helps determine which records should be processed and which should not, especially in scenarios where data arrives late.

Governance Considerations:

Standardization: By making watermarking a standard practice, we ensure that all teams follow the same rules for handling late arriving data.
Avoiding Duplication of Effort: Instead of each team implementing their own lateness semantics, they can rely on established practices and guidelines.

#### Implementation
Steps:

Define Watermark Policies: Establish clear policies on when watermarks are set and how they are used to filter out late records.
Tooling Support: Provide tools that automatically handle watermarking so teams don't have to reinvent the wheel.
Training and Documentation: Ensure all teams are trained and well-versed in using watermarking effectively.

Benefits of Centralized Governance:

#### Cons
istency: Ensures that data processing pipelines behave predictably across different environments.
Scalability: Simplifies scaling operations by reducing the need for custom solutions.
Auditability: Makes it easier to audit and trace data lineage, ensuring compliance with regulations.

#### Trade-offs

While watermarking is powerful, it can introduce complexity in some scenarios where real-time processing requirements are stringent.
It also requires careful design to handle edge cases like extremely late arriving records.

#### Closing Summary
and Common Mistakes

In summary, adopting watermarking as a first-class contract helps maintain consistency and reliability across batch and stream processing pipelines. This approach avoids the governance nightmare of each team independently defining their own lateness semantics. However, it’s crucial to standardize on best practices and provide adequate support through tools and training.

#### Common mistakes
include:

Failing to establish clear policies around watermarking.
Not providing sufficient tooling or documentation for teams.
Overlooking edge cases in the implementation of watermarking.

**Comparison:**

```
Watermark Strategy	Description
For Monotonic Time	Ensures timestamps are in ascending order and handles late data appropriately.
Initial Delay	Waits before starting the watermarking process.
```

**Code:**

```python
# Example Python code snippet using Apache Beam for watermarking

from apache_beam import Pipeline, PipelineOptions
from apache_beam.transforms.window import FixedWindows, TimestampedValue, WatermarkStrategy

def process_element(element):
    # Process each element based on the current watermark
    pass

p = Pipeline(options=PipelineOptions())

# Define a fixed window with a watermark strategy
windowed_pcollection = (
    p
    | 'CreatePCollection' >> beam.Create([('record1', 1), ('record2', 2)])
    | 'WindowIntoFixed' >> beam.WindowInto(
        FixedWindows(30),
        # Use a custom watermarking strategy
        trigger=WatermarkStrategy.for_monotonic_timestamps().with_initial_delay(5).on_element_time(lambda e: e[1]),
        accumulation_mode=beam.window.AccumulationMode.DISCARDING)
    | 'Process' >> beam.Map(process_element)
)

p.run().wait_until_finish()
```

**Example input / output:**

```text
Input:
- Record 1: timestamp = 2023-10-01T12:00:00Z, value = 10
- Record 2: timestamp = 2023-10-01T12:01:00Z, value = 20

Watermark set at 2023-10-01T12:05:00Z (after 5 minutes of the first record's timestamp)

Output:
- Record 1 processed
- Record 2 rejected as it arrived after the watermark (2023-10-01T12:06:00Z)
```

**Data:**

```json
{
  "records": [
    {"id": 1, "value": 10, "timestamp": "2023-10-01T12:00:00Z"},
    {"id": 2, "value": 20, "timestamp": "2023-10-01T12:01:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Record Arrival
4. Is Timestamp < Watermark?
5. Process Data
6. Reject Late Data

