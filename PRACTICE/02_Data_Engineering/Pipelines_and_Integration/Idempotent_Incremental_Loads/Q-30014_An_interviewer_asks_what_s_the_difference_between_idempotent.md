---
type: "practice"
difficulty: "Basic"
---

# An interviewer asks: what's the difference between idempotent and deterministic in the context of a pipeline task, and why does it matter?

**Difficulty:** Basic · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Incremental Loads](<../../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Idempotence and determinism are fundamental concepts in data engineering that ensure reliability and efficiency in pipeline tasks. Idempotence means that executing an operation multiple times has the same effect as executing it once, while determinism ensures that given the same input, a task will always produce the same output. These properties are crucial for building robust and maintainable pipelines.

To understand these concepts fully, let's start with their definitions in the context of data processing.

**Code:**

```python
# Example code to illustrate idempotence and determinism
def process_data(input_data):
    # Idempotent function example
    processed_data = transform_data(input_data)
    return processed_data

def transform_data(data):
    transformed_data = add_timestamp_to_data(data)
    return transformed_data

# Non-idempotent function, as it may change state outside the function scope
def non_idempotent_process(input_data):
    global state_var  # This is a state variable that changes the output on subsequent calls
    processed_data = transform_data(input_data) + " - " + str(state_var)
    state_var += 1
    return processed_data

# Deterministic vs Non-Deterministic Example
def deterministic_process(input_data):
    result = hash(input_data)  # This will always produce the same output for a given input
    return result

def non_deterministic_process(input_data, random_seed=None):
    if random_seed is None:
        import random
        random.seed()  # Uses system time as seed, making it unpredictable
    result = hash(input_data) + random.random()
    return result
```

**Example input / output:**

| Input Data | Deterministic Process Output | Non-Deterministic Process Output (with random_seed=12345) |
|------------|------------------------------|-------------------------------------------------------|
| "data"     | 971086642                    | 971086642.4523809                                      |
| Input Data | Deterministic Process Output | Non-Deterministic Process Output (with random_seed=12345) |
|------------|------------------------------|-------------------------------------------------------|
| "data"     | 971086642                    | 971086642.4523809                                      |

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Input Data
6. Is Process Deterministic?
7. Same Output Each Time
8. Has State Outside FunctionScope?
9. Output May Vary Across Calls
10. Idempotent: Same Output onMultiple Executions
11. idempotent_process(input_data)
12. non_idempotent_process(input_data)

