---
type: "practice"
difficulty: "Expert"
---

# Why should you add a NOT VALID foreign key then VALIDATE it separately when adding constraints to a big table?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with large PostgreSQL tables, adding a NOT VALID foreign key constraint followed by a subsequent VALIDATE CONSTRAINT step can be an effective strategy. This approach allows you to bypass potential performance issues during the initial table population while ensuring data integrity eventually. Let's break down why this method is beneficial and how it works.

**Code:**

```sql
-- Example of adding a NOT VALID constraint
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
REFERENCES customers(id) INITIALLY DEFERRED NOT VALID;

-- Later, validating the constraint to enforce integrity
VALIDATE CONSTRAINT fk_customer_id;
```

