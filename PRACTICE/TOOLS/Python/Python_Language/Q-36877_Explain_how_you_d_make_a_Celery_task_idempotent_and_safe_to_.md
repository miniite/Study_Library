---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd make a Celery task idempotent and safe to retry, including result deduplication and handling a worker that dies after side effect but before ack.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Event Handlers](<../../../../TOOLS/Python/Python_Language/Idempotent_Event_Handlers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a Celery task is both idempotent and safe to retry while also handling result deduplication and dealing with workers that die mid-side-effect, we need to approach this problem from first principles. We'll break it down into several steps:

Understanding the Problem: An idempotent task means the same input will always yield the same output regardless of how many times it's executed. Retrying safely involves ensuring no duplicate work is done and that side effects are handled correctly even if a worker dies before acknowledging completion.

Key Components:

Celery Task Decorator: To mark tasks as idempotent.
Result Deduplication: Ensuring the same task doesn't get executed more than once.
Side Effects Handling: Managing tasks that have side effects and ensuring partial work is rolled back or marked for re-execution.

#### Solution

Celery Task Setup: Define a Celery task with idempotency logic.
Task ID Generation: Use a unique identifier per task execution.
Deduplication Check: Store and check if the task has already been executed using a result backend (Redis, database).
Side Effects Management: Ensure side effects are reversible or idempotent themselves.

Let's walk through these steps in detail:

1. Celery Task Setup
PYTHON
from celery import shared_task
import uuid

@shared_task(bind=True)
def my_idempotent_task(self, param):
    task_id = self.request.id  # Unique identifier for the current task execution

    # Deduplication Logic
    if not self.is_unique(task_id):
        return "Task already executed."

    try:
        result = do_side_effects(param)
        store_result(result)  # Store the result in a result backend
        return f"Task completed successfully with result: {result}"
    except Exception as e:
        handle_exception(e, task_id)
        raise self.retry(exc=e)

def is_unique(task_id):
    # Check if the task has already been executed using a unique key stored in Redis or database
    return not redis.exists(f"{task_id}_executed")

def do_side_effects(param):
    # Simulate side effects that may fail and need to be retried
    # Side effects should ideally be idempotent, e.g., updating a database row
    try:
        # Perform the side effect here
        pass
    except Exception as e:
        raise e

def store_result(result):
    # Store the result in Redis or a database for deduplication checks
    redis.set(f"{task_id}_result", result)

def handle_exception(e, task_id):
    # Handle exceptions and mark the task state appropriately
    self.update_state(state="FAILURE", meta={"error": str(e)})
2. Side Effects Management

For tasks with side effects, ensure they are idempotent or can be safely retried:

PYTHON
def do_side_effects(param):
    try:
        # Example: Updating a row in the database (idempotent if updated properly)
        update_db_row(param)
        return "DB Update Successful"
    except IntegrityError as e:
        raise e  # Retrying will handle this correctly

def update_db_row(param):
    try:
        with transaction.atomic():
            # Atomic block to ensure consistency
            db_object = DBModel.objects.get(id=param)
            db_object.update_field = "new_value"
            db_object.save()
    except IntegrityError as e:
        raise e  # Handle conflict and retry
3. Error Handling and Retrying

Ensure the task can handle partial completion gracefully:

PYTHON
@shared_task(bind=True)
def my_idempotent_task(self, param):
    try:
        result = do_side_effects(param)
        store_result(result)  # Store the result in a result backend
        return f"Task completed successfully with result: {result}"
    except Exception as e:
        handle_exception(e)
        raise self.retry(exc=e)

def handle_exception(e):
    # Update task state and log error
    self.update_state(state="FAILURE", meta={"error": str(e)})

#### Closing Summary

In summary, to make a Celery task idempotent and safe to retry while handling side effects, we need to:

Mark tasks as unique.
Use a result backend for deduplication checks.
Ensure side effects are idempotent or can be safely retried.
Handle exceptions and update the task state appropriately.

#### Common Mistakes/Follow-Ups
Forgetting to handle partial completion, leading to inconsistent states.
Not checking if tasks have already been executed before running them again.
Overlooking side effect reversibility, which can lead to data corruption or loss.

**Comparison:**

| Task ID | Result                      | Side Effect Status |
|---------|-----------------------------|--------------------|
| 12345   | DB Update Successful        | Success            |
| 67890   | Task completed successfully with result: XYZ | Success            |

**Code:**

```python
from celery import shared_task
import uuid
import redis

redis = redis.Redis(host='localhost', port=6379, db=0)

@shared_task(bind=True)
def my_idempotent_task(self, param):
    task_id = self.request.id

    if not is_unique(task_id):
        return "Task already executed."

    try:
        result = do_side_effects(param)
        store_result(result)
        return f"Task completed successfully with result: {result}"
    except Exception as e:
        handle_exception(e, task_id)
        raise self.retry(exc=e)

def is_unique(task_id):
    return not redis.exists(f"{task_id}_executed")

def do_side_effects(param):
    try:
        update_db_row(param)
        return "DB Update Successful"
    except IntegrityError as e:
        raise e

def store_result(result):
    redis.set(f"{task_id}_result", result)

def handle_exception(e, task_id):
    self.update_state(state="FAILURE", meta={"error": str(e)})
```

**Example input / output:**

```text
**Input:**
- Task request with param `42`

**Output:**
- Task execution starts and side effects are performed.
- Result is stored in Redis.
- If a worker dies mid-execution, the task will be retried from where it left off.
- Deduplication checks ensure no duplicate tasks are executed.

**Dry Run Example:**

1. **Task Request:** `my_idempotent_task(42)`
2. **Task Execution:** Side effects performed on DB row with id 42
3. **Result Stored:** Result "DB Update Successful" stored in Redis for task `12345`
```

**Data:**

```json
{
  "redis_connection": {
    "host": "localhost",
    "port": 6379,
    "db": 0
  },
  "task_results": [
    {"id": "12345", "result": "Task completed successfully with result: XYZ"},
    {"id": "67890", "result": "DB Update Successful"}
  ],
  "side_effects": {
    "db_row_id": 1,
    "updated_field": "new_value"
  }
}
```

**Flow:**

1. Yes
2. Start Task
3. Task Unique?
4. Execute Side Effects
5. Store Result in Redis
6. Completion
7. Handle Partial Completion
8. Retry Task

