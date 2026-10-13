---
type: "practice"
difficulty: "Intermediate"
---

# What is idempotency in a data load and why must an ETL job be safe to rerun after a failure?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Late Arriving Dimensions](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Idempotency in a data load ensures that running the same operation multiple times produces the same result as running it once. For an ETL (Extract, Transform, Load) job to be safe to rerun after a failure, it must maintain idempotent behavior. This means that if the job fails midway and is restarted, the outcome should match what would have happened had the job run in one continuous flow from start to finish.

Let's break down why idempotency is crucial for ETL jobs by first understanding the concept of idempotence and then delving into the specific requirements it imposes on data loads.

**Comparison:**

```
Operation	First Execution Result	Second Execution Result
Update Age and Email	{"user123": {"name": "John Doe", "age": 35, "email": "new.email@example.com"}}	{"user123": {"name": "John Doe", "age": 35, "email": "new.email@example.com"}}
```

**Code:**

```python
# Example of an idempotent function in Python
def update_user_profile(user_id, updates):
    # This function ensures that calling it multiple times with the same parameters has no additional effect beyond the first call
    if user_id not in users:
        users[user_id] = {}
    for key, value in updates.items():
        users[user_id][key] = value
```

**Example input / output:**

```

Given an initial state of {"users": {"user123": {"name": "John Doe", "age": 30}}}, applying the updates {"age": 35, "email": "new.email@example.com"} twice should result in:

JSON
Copy
{
  "users": {
    "user123": {
      "name": "John Doe",
      "age": 35,
      "email": "new.email@example.com"
    }
  }
}
```

**Data:**

```json
{
  "users": {
    "user123": {
      "name": "John Doe",
      "age": 30,
      "email": "john@example.com"
    }
  }
}
```

**Flow:**

1. Yes
2. Passes
3. Fails
4. No
5. Start
6. Load Data
7. Is Job Failed?
8. Retry Loading Data
9. Check for Idempotency
10. Continue Processing
11. Handle Error, Retry or Fail
12. End

