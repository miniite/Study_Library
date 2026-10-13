---
type: "practice"
difficulty: "Deep"
---

# How do you decide between a dbt snapshot and an incrementally-built SCD Type 2 fact for tracking customer status history, considering reprocessing and auditability?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Models and Runs](<../../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between using a dbt snapshot or an incrementally-built Slowly Changing Dimension (SCD) Type 2 fact for tracking customer status history in dbt, we need to consider several factors including reprocessing requirements, auditability, and the specific use case. Let’s break down these considerations from first principles.

Firstly, a dbt snapshot is a simple way to track changes by capturing a list of unique identifiers (e.g., customer IDs) that have changed since the last run. This approach works well when you need to ensure that incremental updates are applied correctly without maintaining historical records. On the other hand, an incrementally-built SCD Type 2 fact maintains a full history of changes over time but requires more complex logic and ongoing maintenance.

The core question here is whether we prioritize reprocessing efficiency or detailed auditability. Reprocessing means running your dbt models to update existing data with new information. For snapshot-based models, this process can be faster since only the changed records are processed. However, SCD Type 2 models require more complex logic to handle historical changes and maintain a full history of updates.

Let’s consider some key assumptions:

Data Volume: If your customer base is large and frequently changing, an incrementally-built SCD model might be necessary to avoid overwhelming reprocessing.
Audit Requirements: If strict auditability is required, an SCD Type 2 model ensures that every change in a customer’s status is recorded with a timestamp and old/new values.
Reprocessing Frequency: Frequent reprocessing can make snapshot models more efficient since only incremental changes need to be applied.

Now let's go through the reasoning step-by-step:

Snapshot Model:

#### Pros
Simple implementation, fast reprocessing, easy to maintain.

#### Cons
Does not preserve historical data; requires careful handling of duplicates and new records.
Incremental SCD Type 2 Model:

#### Pros
Preserves history, ensures auditability, supports complex business logic over time.

#### Cons
More complex implementation, slower reprocessing, ongoing maintenance required.

To illustrate the difference, let’s consider a simple example of customer status changes:

#### Example Scenario

Assume we have a table customer_status with columns:

id: Customer ID (primary key)
status: Current status
start_date: Start date for this status

Let's say we want to update the statuses based on some new data.

Snapshot Model Implementation
Create a snapshot model in dbt that captures the updated customer IDs.
Run the model, and it will only process the customers whose IDs have changed since the last run.
SQL
{% snapshot cs_snapshot %}
    select id, status, start_date from {{ source('raw', 'customer_status') }}
    where id in (select id from {{ ref('new_customer_data') }})
{% endsnapshot %}
Incremental SCD Type 2 Model Implementation
Create a model that handles both insert and update logic.
Maintain historical records of status changes with start/end dates.
SQL
with customer_updates as (
    select * from {{ source('raw', 'customer_status') }}
),

new_data as (
    select * from {{ ref('new_customer_data') }}
)

select
    id,
    new_data.status as new_status,
    current_timestamp() as start_date,
    lag(cs_snapshot.end_date, 1) over (partition by id order by cs_snapshot.start_date desc) as end_date
from customer_updates

#### Common Mistakes
and Follow-Ups
Ignoring Historical Context: Always consider the full history of changes. Snapshots can miss important context if historical data is not preserved.
Overcomplicating Snapshots: Sometimes, a simple snapshot might suffice for less critical use cases.

#### Performance Considerations
For very large datasets or high-frequency updates, SCD Type 2 models require more careful tuning to maintain performance.

#### Closing Summary

In summary, the choice between using dbt snapshots and incrementally-built SCD Type 2 fact tables depends on your specific requirements for reprocessing efficiency versus auditability. For frequent changes and strict auditing needs, an SCD Type 2 model is often preferred despite its complexity. However, if you can tolerate less detailed records and need faster reprocessing, a snapshot approach may be more appropriate.

Self-Check

The answer covers the core concepts of both approaches, their trade-offs, and key considerations comprehensively.

**Code:**

```sql
-- Example dbt snapshot model
{% snapshot cs_snapshot %}
    select id, status, start_date from {{ source('raw', 'customer_status') }}
    where id in (select id from {{ ref('new_customer_data') }})
{% endsnapshot %}

-- Example SCD Type 2 model logic
with customer_updates as (
    select * from {{ source('raw', 'customer_status') }}
),

new_data as (
    select * from {{ ref('new_customer_data') }}
)

select
    id,
    new_data.status as new_status,
    current_timestamp() as start_date,
    lag(cs_snapshot.end_date, 1) over (partition by id order by cs_snapshot.start_date desc) as end_date
from customer_updates
```

**Example input / output:**

```text
Snapshot Model:
Input:
- customer_status (old)
- new_customer_data

Output:
- Updated customer_status with only active changes.

Incremental SCD Type 2 Model:
Input:
- customer_updates
- new_data

Output:
- Updated SCD table with historical records of status changes.
```

**Data:**

```json
{
  "customer_status": [
    {"id": 1, "status": "Active", "start_date": "2023-01-01"},
    {"id": 2, "status": "Inactive", "start_date": "2023-01-01"}
  ],
  "new_customer_data": [
    {"id": 1, "status": "Suspended", "start_date": "2023-04-01"},
    {"id": 3, "status": "Active", "start_date": "2023-04-01"}
  ]
}
```

