---
type: "practice"
difficulty: "Intermediate"
---

# How does SCD Type 2 preserve history and what extra columns does it typically add?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: SCD Type 2, or Slowly Changing Dimension Type 2, is a technique used in data warehousing to manage historical changes in dimension tables without deleting old records. It typically adds extra columns like EffectiveDate, EndDate, and sometimes IsCurrent to track the version of the record.

#### Triage
The question asks about SCD Type 2 and its implementation details, focusing on how it preserves history and what additional columns are used. We need to understand that this is a method for handling changing data in dimensional modeling. Specifically, we will detail how these extra columns work together to manage historical changes without altering the original records.

First Principles: SCD Type 2 is about keeping historical versions of dimension records by appending new rows rather than updating existing ones. This approach ensures that the entire history of a dimension record is preserved in the data warehouse. The key columns added are:

EffectiveDate/StartDate: Marks the start of this version of the record.
EndDate: Marks when this version ends or will be considered inactive (NULL until explicitly set).
IsCurrent (optional): A boolean field that helps identify which is the current active version.

Let's walk through a detailed example to see how these columns interact:

Suppose we have a customer dimension table with an id, name, and address. Initially, a new record might look like this:

PLAINTEXT
id | name    | address   | EffectiveDate | EndDate    | IsCurrent
-- | ------- | --------- | ------------- | ---------- | --------
1  | John Doe| 123 Main  | 2020-01-01    | NULL       | 1

Now, if the customer moves to a new address on 2021-05-01, we add another row:

PLAINTEXT
id | name    | address   | EffectiveDate | EndDate    | IsCurrent
-- | ------- | --------- | ------------- | ---------- | --------
1  | John Doe| 456 Oak   | 2021-05-01    | NULL       | 0

The previous record's EndDate is set to the current date to indicate it ended:

PLAINTEXT
id | name    | address   | EffectiveDate | EndDate    | IsCurrent
-- | ------- | --------- | ------------- | ---------- | --------
1  | John Doe| 123 Main  | 2020-01-01    | 2021-04-30 | 0

Finally, we set the IsCurrent flag to 1 for the current version:

PLAINTEXT
id | name    | address   | EffectiveDate | EndDate    | IsCurrent
-- | ------- | --------- | ------------- | ---------- | --------
1  | John Doe| 456 Oak   | 2021-05-01    | NULL       | 1

#### Closing Summary
SCD Type 2 is a robust method for handling slowly changing dimensions by maintaining the historical context of data. It uses EffectiveDate, EndDate, and optionally IsCurrent to preserve all versions of a record.

#### Common Mistakes/Follow-Ups
A common mistake is not understanding that setting IsCurrent only for one row at a time can lead to confusion. Always ensure that only the most current version has its IsCurrent set to 1 while others are set to 0 or NULL.

Another follow-up could be about managing more complex changes, like when attributes change but the record's identity remains the same—this would require additional logic and possibly SCD Type 3 (or hybrid types).

**Comparison:**

| Column | Description |
|--------|-------------|
| id     | Unique identifier for the customer. |
| name   | Customer name. |
| address| Customer address. |
| effective_date | The start date of this record version. |
| end_date    | The end date of this record version (NULL if current). |
| is_current  | Boolean flag indicating if this record is the current active version. |

**Code:**

```sql
-- Example SQL for adding new records with SCD Type 2
INSERT INTO customers_dim
(id, name, address, effective_date, end_date, is_current)
VALUES
(1, 'John Doe', '456 Oak', '2021-05-01', NULL, 1);

UPDATE customers_dim
SET end_date = '2021-04-30'
WHERE id = 1 AND name = 'John Doe' AND address = '123 Main';
```

**Example input / output:**

```text
Input:
id = 1, name = 'John Doe', address = '456 Oak', effective_date = '2021-05-01', end_date = NULL, is_current = 1

Output:
Record with id=1, name='John Doe' has new version added.
Previous record's end_date set to 2021-04-30
```

**Data:**

```json
{
  "id": 1,
  "name": "John Doe",
  "address": "456 Oak",
  "effective_date": "2021-05-01",
  "end_date": null,
  "is_current": true
}
```

**Flow:**

1. Check for Existing Record
2. (Update EndDate andIsCurrent)
3. Set IsCurrent to 0
4. Insert New Record withEffectiveDate
5. End

