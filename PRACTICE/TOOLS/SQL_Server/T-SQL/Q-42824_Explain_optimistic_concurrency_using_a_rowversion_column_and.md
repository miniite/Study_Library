---
type: "practice"
difficulty: "Deep"
---

# Explain optimistic concurrency using a rowversion column and how the application detects a conflicting update.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ MERGE Pitfalls and Error Handling](<../../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic Concurrency is a technique used to ensure data consistency in concurrent updates by tracking changes without actually locking rows until an update attempt. In T-SQL, this is often implemented using a rowversion or timestamp column. When an application retrieves a row, it stores the value of the rowversion column. Upon attempting to update the row, it checks if the current rowversion matches the one stored; if they don't match, another concurrent transaction has updated the row since its last retrieval.

Let's dive into how this works in more detail.

**Comparison:**

```
EmployeeID	Name	Salary	RowVersion
1	Alice	5000.00	9223372036854775807
```

**Code:**

```sql
-- Example of a table with a rowversion column
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Name NVARCHAR(100),
    Salary DECIMAL(18, 2),
    -- Rowversion/timestamp column for optimistic concurrency
    [RowVersion] TIMESTAMP
);
```

**Example input / output:**

```

When an application retrieves an employee record, it stores the rowversion value. Here’s a simplified example of how this might look in a C# application using ADO.NET:

CSHARP
Copy
// Retrieve the row
var sql = "SELECT * FROM Employees WHERE EmployeeID = @EmployeeID";
using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
    var command = new SqlCommand(sql, connection);
    command.Parameters.AddWithValue("@EmployeeID", 1);

    using (var reader = command.ExecuteReader())
    {
        if (reader.Read())
        {
            // Store the rowversion value
            long rowVersion = reader.GetLong(3); // Assuming RowVersion is the fourth column

            // Later, when updating the row
            var updateSql = "UPDATE Employees SET Salary = 5100.00 WHERE EmployeeID = @EmployeeID AND [RowVersion] = @RowVersion";
            command.CommandText = updateSql;
            command.Parameters.Clear();
            command.Parameters.AddWithValue("@EmployeeID", 1);
            command.Parameters.AddWithValue("@RowVersion", rowVersion);

            int rowsAffected = command.ExecuteNonQuery();

            if (rowsAffected == 0)
            {
                // Another transaction updated the row in the meantime
                Console.WriteLine("Concurrency conflict detected.");
            }
        }
    }
}
```

**Data:**

```json
{
  "EmployeeID": 1,
  "Name": "Alice",
  "Salary": 5000.00,
  "RowVersion": 9223372036854775807 -- This is the initial timestamp value
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Retrieve row
6. Is rowversion stored?
7. Store rowversion
8. Fetch and store rowversion
9. Update with stored rowversion
10. Was update successful?
11. Row updated successfully
12. Concurrency conflict detected

