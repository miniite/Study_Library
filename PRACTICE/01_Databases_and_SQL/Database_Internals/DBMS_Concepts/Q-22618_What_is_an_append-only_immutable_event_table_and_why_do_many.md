---
type: "practice"
difficulty: "Deep"
---

# What is an append-only immutable event table and why do many pipelines prefer never updating existing rows?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An append-only immutable event table in a data pipeline context refers to a design pattern where each new piece of data (event) is inserted into a new row, and historical changes are recorded as additional events rather than updating existing rows. This approach provides several benefits that many pipelines prefer, including ease of rollback, auditability, and consistency.

The core idea here is that in an append-only system, every change or event is treated as a new piece of data, which can be inserted at the end of the table without altering any existing entries. This contrasts with traditional row-based updating, where modifying an existing row might disrupt historical integrity and introduce complexity.

To fully understand why many pipelines prefer this design, let's start from first principles:

#### Triage
What is Being Probed

The interviewer is probing your understanding of data storage paradigms, the trade-offs between different database designs, and how these choices impact data integrity and manageability. Key assumptions include:

Understanding of basic SQL operations like INSERT vs UPDATE.
Knowledge of historical data requirements and their implications on data management.
Familiarity with common issues in data pipelines such as rollbacks and consistency checks.

#### First-Principles Solution
Data Integrity and Historical Tracking: In an append-only design, each event is recorded as a new row, preserving the integrity of the historical data. This makes it easier to track changes over time without worrying about losing information or accidentally reverting history.
Simplicity in Rollbacks: If something goes wrong, you can simply discard the most recent events and start fresh, making rollbacks straightforward and less error-prone compared to updating existing rows, which could introduce complex cascading effects.

#### Cons
istency Checks: Append-only tables are simpler for consistency checks because they don't involve modifying data that might be read by other systems or processes. This reduces the risk of unintended side-effects during updates.
Auditing and Compliance: Immutable event logs make it easier to audit changes, which is critical in regulated industries where data integrity must be verifiable.
Detailed Explanation
Historical Consistency: In traditional databases, updating a row can affect not just the current state but also any views or reports that depend on that row's historical state. This introduces complexity and potential bugs.
Read Performance: Append-only designs often have better read performance because new data is always at the end of the table, which can be optimized for faster access using efficient indexing techniques like append-only indexes in some database systems.
Cost Considerations: While appending data might increase storage costs over time, modern cloud storage solutions offer cost-effective ways to manage large volumes of immutable data. The trade-off here is between immediate cost savings from row updates versus long-term operational simplicity and auditability.

#### Closing Summary

In summary, an append-only immutable event table offers a robust solution for maintaining historical integrity, simplifying rollbacks, enhancing consistency checks, and supporting easier audits and compliance. While there might be storage costs associated with keeping all events forever, the benefits in terms of data reliability and manageability often outweigh these concerns.

#### Common Mistakes/Follow-Ups
Misunderstanding Row Updates: Failing to recognize that row updates can introduce complexity and potential bugs.
Neglecting Historical Data Needs: Not considering the importance of maintaining a complete history for auditing purposes, especially in regulated industries.
Overlooking Cost Implications: Ignoring the long-term storage costs associated with immutable data but focusing only on short-term operational simplicity.

**Comparison:**

| Event Table Schema |
|-------------------|
| timestamp         | datetime       |
| user_id           | varchar(255)   |
| action            | enum('login', 'logout') |
| Historical Data Tracking Table (Optional for Audit) |
|--------------------------------------------------|
| event_id      | int          |
| timestamp     | datetime     |
| user_id       | varchar(255) |
| old_action    | enum         |
| new_action    | enum         |

**Code:**

```sql
-- Example of an append-only table insert query
INSERT INTO event_table (timestamp, user_id, action)
VALUES ('2023-10-01 12:00:00', 'user_123456', 'login');

-- Contrast with a traditional update query
UPDATE event_table SET action = 'logout' WHERE timestamp = '2023-10-01 12:00:00';
```

**Example input / output:**

```text
Input (Event):
user_id = 'user_123456'
action = 'login'

Output (Append-only Table):
timestamp: '2023-10-01T12:00:00Z'
user_id: 'user_123456'
action: 'login'

Input (Event for same user, different action):
user_id = 'user_123456'
action = 'logout'

Output (Append-only Table):
timestamp: '2023-10-01T12:01:00Z'
user_id: 'user_123456'
action: 'logout'
```

**Data:**

```json
{
  "user_id": "user_123456",
  "events": [
    { "timestamp": "2023-10-01T12:00:00Z", "action": "login" },
    { "timestamp": "2023-10-01T12:01:00Z", "action": "logout" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event Occurs
4. Is Event Immutable?
5. Insert New Row
6. Update Existing Row
7. Record Historical Changes
8. Historical Data at Risk

