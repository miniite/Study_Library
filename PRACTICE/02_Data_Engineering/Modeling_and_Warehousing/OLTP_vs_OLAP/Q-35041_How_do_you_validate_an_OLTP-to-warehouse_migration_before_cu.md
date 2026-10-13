---
type: "practice"
difficulty: "Deep"
---

# How do you validate an OLTP-to-warehouse migration before cutover — describe your row-count, checksum, and business-metric reconciliation plan and what tolerance you'd accept.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When validating an OLTP (Online Transaction Processing) to warehouse migration before a cutover, it's crucial to ensure that the data in the new system accurately reflects the state of the original OLTP database at the time of migration. This involves a comprehensive plan for row-count validation, checksum verification, and business-metric reconciliation.

The core idea is to use these checks as a safeguard against potential data discrepancies, ensuring high confidence in the integrity and consistency of the migrated data before committing to the cutover process. We'll break down each step from first principles and discuss the assumptions and tolerances involved.

**Code:**

```python
# Example Python function for checksum calculation using SHA-256
import hashlib

def calculate_checksum(data):
    sha256 = hashlib.sha256()
    sha256.update(data.encode('utf-8'))
    return sha256.hexdigest()

# Example row-count validation script
def validate_row_count(oltp_cursor, warehouse_cursor, table_name):
    oltp_rows = oltp_cursor.execute(f"SELECT COUNT(*) FROM {table_name}").fetchone()[0]
    warehouse_rows = warehouse_cursor.execute(f"SELECT COUNT(*) FROM {table_name}").fetchone()[0]
    return (oltp_rows == warehouse_rows)

# Example business-metric reconciliation script
def reconcile_business_metric(oltp_cursor, warehouse_cursor, metric_sql):
    oltp_value = float(oltp_cursor.execute(metric_sql).fetchone()[0])
    warehouse_value = float(warehouse_cursor.execute(metric_sql).fetchone()[0])
    return abs(oltp_value - warehouse_value) < tolerance
```

**Example input / output:**

```
Example input for row-count validation

Input:

OLTP query: "SELECT COUNT(*) FROM customers"
Warehouse query: "SELECT COUNT(*) FROM customers"

Output:

OLTP result: 10,000 rows
Warehouse result: 10,000 rows

Conclusion: Row counts match.

CODE
Copy

```plaintext
# Example input for business-metric reconciliation
Input:
- OLTP query: "SELECT SUM(sales_amount) FROM orders"
- Warehouse query: "SELECT SUM(sales_amount) FROM orders"

Output:
- OLTP result: $5,000,000.00
- Warehouse result: $5,000,123.45

Conclusion: Metric difference is within tolerance.
```

**Flow:**

1. Yes
2. No
3. Start Validation
4. Is OLTP row count =Warehouse row count?
5. Log Pass and Continue
6. Identify and InvestigateDiscrepancies
7. Reconcile Business Metrics(Optional)
8. End Validation

