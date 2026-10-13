---
type: "practice"
difficulty: "Deep"
---

# How do you handle modeling of soft-deletes, re-activations, and tombstones across SCD dimensions and Vault satellites so history reflects reality, not source quirks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Vault](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle modeling of soft-deletes, re-activations, and tombstones in a Data Warehouse environment using Kimball/Inmon/Data Vault techniques, you need to ensure that your data model reflects reality accurately while maintaining historical integrity. This involves understanding the source system quirks, deciding on the appropriate data modeling strategy, and implementing robust ETL processes.

Firstly, let’s clarify what we’re dealing with:

Soft Deletes: Records are marked as deleted but not physically removed.
Re-Activations: Soft-deleted records that are brought back into active status.
Tombstones: A record is permanently deleted from the source system and needs to be reflected in the data warehouse.

#### Triaging the Question

The core of this question lies in ensuring that your data model accurately represents the business processes, even when dealing with anomalies or edge cases in the source systems. The interviewer is probing your ability to handle historical data integrity, manage concurrency issues, and maintain a consistent view of reality across time.

Key assumptions include:

Source systems may have quirks like soft-deletes.
You need to capture both current state and historical states.
Re-activations and tombstones must be handled correctly without causing data inconsistencies.

#### First-Principles Solution

Identify the Business Requirements:

Understand why soft-deletes, re-activations, and tombstones are necessary in your business context.
Determine if these processes need to be reflected accurately in all fact tables or just specific ones.

Choose the Appropriate Data Vault Strategy:

For Kimball’s star schema, you might use Slowly Changing Dimensions (SCDs) of type 1, 2, and 3.
For Inmon’s dimensional modeling approach, consider using a combination of SCDs and transactional history.
With Data Vault, use JUNKs to handle tombstones, HUBs for primary keys, and LINKS for relationships.

ETL Processes:

Develop ETL processes that can handle soft-deletes by updating the dimension table with a DELETE flag or a valid-to date.
For re-activations, ensure updates reflect the change in status accurately.
Implement tombstones by creating a separate FACT_TOMBSTONES table or tagging records in the primary fact table.

Historical Data Integrity:

Maintain a history of all changes to ensure that your data warehouse reflects the actual business processes over time.
Use versioning and timestamps to manage concurrent updates.

Validation and Testing:

Validate ETL processes through unit tests, integration testing, and regression testing.
Ensure that historical reports are accurate by verifying them against known source data.

#### Closing Summary

In summary, handling soft-deletes, re-activations, and tombstones requires a combination of sound business understanding, appropriate data modeling techniques (Kimball/Inmon/Data Vault), robust ETL processes, and thorough testing. The goal is to maintain historical integrity while ensuring that the data warehouse accurately reflects reality.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include:

Failing to properly handle concurrent updates.
Not capturing sufficient history leading to data inconsistencies.
Ignoring source system quirks during design.

#### Follow-ups
might include questions about specific tools used for ETL, how you handle large volumes of soft-deletes, and strategies for maintaining performance in the presence of frequent updates.

**Comparison:**

```
records
sales
HUB_CUST
int
customer_id
PK
string
first_name
string
last_name
LINK_CUSTOMER_CHANGES
int
link_id
PK
int
customer_id
FK
date
valid_from_date
date
valid_to_date
deletion
events
FACT_SALES
involving
customer
```

**Code:**

```sql
-- Example of a SQL query to handle SCD type 2 updates

UPDATE dim_customer
SET is_deleted = TRUE,
    valid_to_date = CURRENT_DATE
WHERE customer_id = '12345' AND is_deleted = FALSE;

INSERT INTO dim_customer (customer_id, first_name, last_name, valid_from_date, valid_to_date)
VALUES ('12345', 'John', 'Doe', CURRENT_DATE, NULL);
```

**Example input / output:**

```text
Input:
- Source system sends a soft-delete for customer with ID '12345'.
Output:
- dim_customer table marks the record as deleted.
- link_customer_changes table logs the deletion event.
- All related fact tables update to reflect the change in status.

Request:
- A reactivation request is received for customer '12345'.
Response:
- dim_customer table updates the valid_to_date field and sets is_deleted to FALSE.
```

**Data:**

```json
{
  "hub_customers": {
    "customer_id": 1,
    "first_name": "John",
    "last_name": "Doe"
  },
  "link_customer_changes": {
    "customer_id": 1,
    "change_type": "delete",
    "valid_from_date": "2023-10-01"
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Soft-Delete Request
4. Updatedim_customer.mark_deleted
5. Check for Re-Activations
6. Reactivate indim_customer.set_active
7. Record Deletion inlink_customer_changes
8. Update Fact Tables
9. Record Deletion Event
10. Receive Reactivation Request
11. Updatedim_customer.unmark_deleted
12. Reactivate in fact_tables

