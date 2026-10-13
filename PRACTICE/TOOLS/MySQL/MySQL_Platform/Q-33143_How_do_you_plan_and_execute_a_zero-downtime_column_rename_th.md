---
type: "practice"
difficulty: "Deep"
---

# How do you plan and execute a zero-downtime column rename that a running application still reads/writes, using the expand-contract migration pattern?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve a zero-downtime column rename in MySQL while ensuring an application can continue to read and write data during the operation, we will use the expand-contract migration pattern. This involves adding a new column with the desired name, updating the application to start using the new column for reads and writes, then removing the old column once all necessary changes are made. The key steps involve careful planning of the database schema, application logic, and testing.

The core idea is to minimize disruption by gradually shifting traffic from the old column to the new one without any sudden change that could affect ongoing operations.

**Code:**

```sql
-- Example SQL for adding a new column with the desired name
ALTER TABLE users ADD COLUMN new_column_name VARCHAR(255);

-- Sample application code to update logic to use new_column_name
// Assuming this is part of your application's database interaction layer
function updateUser(user) {
    // Before migration:
    // db.query(`UPDATE users SET column_name = ${user.column_name} WHERE id = ${user.id}`);

    // During/After migration:
    db.query(`UPDATE users SET new_column_name = ? WHERE id = ?`, [user.new_column_name, user.id]);
}
```

**Example input / output:**

```text
// Before Migration
Input: db.query(`UPDATE users SET column_name = 'new_value' WHERE id = 1`);
Output: Success (but the data remains in the old format)

// During Migration
Input: db.query(`UPDATE users SET new_column_name = 'updated_value' WHERE id = 2`);
Output: Success, and now both columns will contain updated values

// After Migration
Input: db.query(`ALTER TABLE users DROP COLUMN column_name;`);
Output: Success (old column dropped)
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "column_name": "old_value",
      "new_column_name": null
    },
    {
      "id": 2,
      "column_name": "another_value",
      "new_column_name": "updated_value"
    }
  ]
}
```

