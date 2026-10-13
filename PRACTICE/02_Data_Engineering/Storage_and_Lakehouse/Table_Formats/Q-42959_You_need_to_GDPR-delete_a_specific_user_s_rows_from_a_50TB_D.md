---
type: "practice"
difficulty: "Intermediate"
---

# You need to GDPR-delete a specific user's rows from a 50TB Delta table. Walk through how you'd do it efficiently and what VACUUM does afterward.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently GDPR-delete a specific user's rows from a 50TB Delta Lake table, we'll use Delta Lake’s powerful DELETE functionality combined with Hudi's deleteWhere mechanism if necessary. The key steps involve crafting the correct SQL query to delete the relevant data, ensuring that these deletions are consistent and transactionally safe. After executing the deletion, we’ll run a VACUUM operation to reclaim storage.

The essence of this task is twofold: first, performing an efficient row deletion using Delta Lake or Hudi; second, managing the resulting metadata and storage optimization via VACUUM.

**Code:**

```python
# Example Python code snippet for deleting rows based on a condition in Delta Lake
from delta import DeltaTable

def gdpr_delete(delta_table_path, user_id):
    # Step 1: Open the Delta table using Delta API
    delta_table = DeltaTable.forPath(spark, delta_table_path)

    # Step 2: Perform DELETE operation to remove specific user's rows
    delta_table.delete("user_id = 'user_to_delete'")
```

**Example input / output:**

```text
No worked input->output examples needed here, as the example focuses on the process rather than a specific dataset.
```

**Flow:**

1. Delta Lake
2. Hudi
3. Start
4. Is Delta Lake or Hudi?
5. Execute DELETE query
6. Use deleteWhere method tospecify condition
7. Commit transaction andupdate metadata
8. VACUUM operation forstorage optimization
9. End

