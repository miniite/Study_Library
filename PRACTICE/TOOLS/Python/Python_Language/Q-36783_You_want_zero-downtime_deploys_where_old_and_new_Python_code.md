---
type: "practice"
difficulty: "Expert"
---

# You want zero-downtime deploys where old and new Python code run simultaneously against the same DB. What backward/forward-compatibility rules do you enforce for schema and serialization?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a zero-downtime deploy scenario, ensuring that both old and new versions of your application can coexist and communicate with each other is crucial. This requires careful management of database schemas and data serialization formats to ensure backward- and forward-compatibility. For Python applications, this involves several key steps:

Schema Compatibility: Ensure the schema changes are gradual and do not break existing code.
Serialization Consistency: Use compatible libraries for handling data interchange between versions.

Let's dive into these in more detail from first principles.

**Comparison:**

```
User Model	Old Schema (User)	New Schema (UserV2)
ID	AutoField	AutoField
Name	CharField(100)	CharField(100, null=False)
Is Active	-	BooleanField(default=True)
```

**Code:**

```python
# Example of a simple ORM model before refactoring
class User(models.Model):
    id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=100)

# After refactoring with a new field for versioning
class UserV2(models.Model):
    id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=100, null=False)  # Ensures non-null constraint
    is_active = models.BooleanField(default=True)        # New field to mark active users
```

**Example input / output:**

```

For a migration from User to UserV2, you might start with an initial state where all users are active and then update the state during the rollout.

PYTHON
Copy
# Initial data migration script
def migrate_users():
    for user in User.objects.all():
        new_user = UserV2(id=user.id, name=user.name)
        # Set is_active to True for existing users as a default
        new_user.is_active = True
        new_user.save()

migrate_users()
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "is_active": true},
    {"id": 2, "name": "Bob"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a new schema?
5. Migrate old data
6. Rollout new version
7. Monitor and manage rollbacksif needed
8. End
9. Continue with current schema

