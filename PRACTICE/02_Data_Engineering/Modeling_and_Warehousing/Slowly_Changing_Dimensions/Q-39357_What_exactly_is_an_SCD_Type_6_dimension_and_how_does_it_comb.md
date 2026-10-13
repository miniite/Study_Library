---
type: "practice"
difficulty: "Intermediate"
---

# What exactly is an SCD Type 6 dimension and how does it combine Types 1, 2 and 3 in one table?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Slowly Changing Dimension (SCD) types are used to handle changes in data over time in a dimensional modeling context. SCD Type 6 combines elements of Types 1, 2, and 3 by creating a new record with a unique surrogate key while preserving historical records and the original surrogate key.

#### Triage

The question asks for an explanation of what SCD Type 6 is and how it integrates features from SCD Types 1, 2, and 3. Specifically, we need to understand that each type addresses changes in dimensions differently:

Type 1: Overwrites the existing record with a new one.
Type 2: Adds a version column to keep historical records intact.
Type 3: Uses both surrogate keys and a version column.

#### First-Principles Solution
What is SCD Type 6?

SCD Type 6 involves creating a new surrogate key for each change in the dimension, thereby maintaining history while preserving the original surrogate key. This approach combines the benefits of Types 1, 2, and 3 by ensuring that historical data remains accessible.

Combining SCD Types:
Preserving Historical Data (Type 2): Type 6 retains the versioning mechanism to keep track of changes over time.
Maintaining Original Surrogate Key (Type 1 & 2): The original surrogate key is retained for consistency and reference, while a new surrogate key is generated for each change.
Handling New Data (Type 3): A unique surrogate key allows tracking the history of the dimension while adding new records.

#### Example Scenario

#### Cons
ider a customer table where you want to record changes in a customer's address. Initially, you have a single record with a surrogate_key and a version_number.

Initial Record:

PLAINTEXT
surrogate_key: 12345
version_number: 1
address: '123 Main St'

Address Change to New Address:

Create a new record with a different surrogate_key.
Increment the version_number of the original record.
PLAINTEXT
# Original Record (unchanged)
surrogate_key: 12345
version_number: 2

# New Record (address change)
surrogate_key: 67890
version_number: 1
address: '456 Elm St'
Key Implementation Points:
Surrogate Key Management: Use a unique surrogate_key for each new record to maintain history.
Version Numbering: Maintain a version number in the original record to track changes.
Data Integrity: Ensure that the new records and historical versions coexist without conflicts.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

SCD Type 6 provides a comprehensive approach by combining elements from Types 1, 2, and 3. It ensures historical data is preserved while maintaining a consistent reference to the original record. Common mistakes include failing to properly manage surrogate keys or version numbers, leading to data integrity issues.

#### Common Mistakes/Follow-Ups
Forgetting to Increment Version Numbers: This can lead to losing track of history.
Incorrect Surrogate Key Generation: Using the same surrogate_key for new records can cause confusion and conflicts.
Ignoring Performance Considerations: Maintaining multiple version numbers and surrogate keys can impact performance if not optimized.

**Comparison:**

```
table CustomerDim {
    surrogate_key INT PRIMARY KEY,
    original_surrogate_key INT,
    version_number INT,
    address VARCHAR(255),
    effective_date DATE,
    expiration_date DATE
}
```

**Code:**

```sql
-- Example SQL to implement SCD Type 6
CREATE TABLE customer_dim (
    surrogate_key INT PRIMARY KEY,
    original_surrogate_key INT,
    version_number INT,
    address VARCHAR(255),
    effective_date DATE,
    expiration_date DATE
);

-- Insert initial record
INSERT INTO customer_dim (surrogate_key, original_surrogate_key, version_number, address)
VALUES (12345, 12345, 1, '123 Main St');

-- Update record with new address
WITH latest_version AS (
    SELECT MAX(version_number) as max_version FROM customer_dim WHERE surrogate_key = 12345 AND address = '123 Main St'
)
INSERT INTO customer_dim (surrogate_key, original_surrogate_key, version_number, address, effective_date, expiration_date)
SELECT
    (SELECT COALESCE(MAX(surrogate_key) + 1, 67890) FROM customer_dim), -- Generate new surrogate key
    12345,
    next_version.version_number + 1 as version_number,
    '456 Elm St',
    CURRENT_DATE,
    NULL
FROM latest_version next_version;
```

**Example input / output:**

```text
Initial Record:
surrogate_key: 12345
original_surrogate_key: 12345
version_number: 1
address: '123 Main St'
effective_date: '2023-01-01'

Updated Record:
surrogate_key: 67890
original_surrogate_key: 12345
version_number: 1
address: '456 Elm St'
effective_date: '2023-02-01'
```

**Data:**

```

{ "surrogate_key": 12345, "original_surrogate_key": 12345, "version_number": 1, "address": "123 Main St", "effective_date": "2023-01-01", "expiration_date": null }

CODE
Copy

```json
{
  "surrogate_key": 67890,
  "original_surrogate_key": 12345,
  "version_number": 1,
  "address": "456 Elm St",
  "effective_date": "2023-02-01",
  "expiration_date": null
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Initial Record
6. Is Address Changed?
7. Keep Record
8. Generate New Surrogate Key?
9. Create New Record withVersion Number
10. Update Existing Record

