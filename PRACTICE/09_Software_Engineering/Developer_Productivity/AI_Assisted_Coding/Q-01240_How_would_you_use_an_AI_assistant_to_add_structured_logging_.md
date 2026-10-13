---
type: "practice"
difficulty: "Intermediate"
---

# How would you use an AI assistant to add structured logging (JSON, correlation IDs) across a service consistently?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompting for Validation and Explanations](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Validation_and_Explanations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how an AI assistant like Copilot can be used to add structured logging with JSON and correlation IDs consistently across a service, let's break down the problem from first principles. We'll start by understanding what structured logging is, why it’s important, and then see how an AI tool can assist in implementing this effectively.

First, structured logging involves using a consistent format (like JSON) to log data in a way that is easily machine-readable and searchable. This helps in better monitoring, debugging, and troubleshooting applications, especially in distributed systems where logs might come from multiple sources. The key challenge here is ensuring consistency across the service to avoid confusion or errors.

#### Triaging the Question

The question essentially asks how we can leverage an AI tool like Copilot to add structured logging with JSON and correlation IDs consistently across a service. Here are the assumptions:

We're working on a backend service that needs enhanced logging for better observability.
The service is expected to operate in a distributed environment, which means logs might come from multiple sources.

#### Cons
istency is crucial to ensure reliable monitoring and debugging.

#### First Principles Solution
Identify Logging Points: Determine where logging should occur within the application lifecycle—before and after critical operations, at decision points, or when errors happen.
Define a Structured Format: Use JSON to define a consistent log format that includes fields like timestamp, level, correlation_id, message, etc.
AI-Assisted Code Insertion: Utilize Copilot to automate the insertion of logging statements at key points in the code.
Step-by-Step Implementation

Set Up Copilot Integration:

Ensure your development environment supports AI tools like Copilot and is properly configured for it.

Define a Logging Template:

JSON
{
  "timestamp": "{{now}}",
  "level": "{{log_level}}",
  "correlation_id": "{{correlation_id}}",
  "message": "{{message}}"
}

Integrate Copilot for Automated Logging:

Use Copilot to insert logging statements at critical points in your code.
For example, if a function is called, use Copilot to automatically generate the following snippet:
PYTHON
# Before the function call
log.info(json.dumps({
    "timestamp": datetime.now().isoformat(),
    "level": "INFO",
    "correlation_id": "1234567890",
    "message": f"Calling {function_name}"
}))

# Inside the function (if needed)
if some_condition:
    log.error(json.dumps({
        "timestamp": datetime.now().isoformat(),
        "level": "ERROR",
        "correlation_id": "1234567890",
        "message": f"Condition not met: {some_condition}"
    }))

# After the function call
log.info(json.dumps({
    "timestamp": datetime.now().isoformat(),
    "level": "INFO",
    "correlation_id": "1234567890",
    "message": f"Function returned: {return_value}"
}))

Ensure Consistency:

Use Copilot to consistently apply the same logging template across similar functions.
Introduce a code review process to ensure all new changes follow this standard.

#### Closing Summary

By leveraging an AI assistant like Copilot, we can automate the insertion of structured logging statements into our service. This approach ensures consistent and standardized logging practices, making it easier to monitor and debug issues in a distributed system. The key is defining clear templates and leveraging automation tools to enforce these standards across the codebase.

#### Common Mistakes/Follow-Ups
Inconsistent Logging: Ensure that all parts of the code follow the same logging pattern.
Performance Overhead: Be mindful of performance impacts, especially in high-throughput systems where excessive logging can become a bottleneck.
Debugging Difficulty: Too much or unstructured data can make logs difficult to parse. It’s important to keep them concise and relevant.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Example of Logging Workflow with Copilot
    dateFormat XYYYY-MM-DD
    section Initial Setup
    Copilot Integration: date("2023-10-05")
    Define Log Template: date("2023-10-06")
    section Implementation
    Insert Logging Statements: dateRange("2023-10-07", "2023-10-10")
    Review and Iterate: date("2023-10-11")
    section Maintenance
    Ensure Consistency: date("2023-10-12")
    Performance Testing: date("2023-10-15")
```

**Code:**

```python
# Example Python Logging Statement with Copilot Insertion
def some_function(some_condition):
    log.info(json.dumps({
        "timestamp": datetime.now().isoformat(),
        "level": "INFO",
        "correlation_id": "1234567890",
        "message": f"Calling {some_function.__name__}"
    }))

    if some_condition:
        log.error(json.dumps({
            "timestamp": datetime.now().isoformat(),
            "level": "ERROR",
            "correlation_id": "1234567890",
            "message": f"Condition not met: {some_condition}"
        }))

    result = some_calculations()
    log.info(json.dumps({
        "timestamp": datetime.now().isoformat(),
        "level": "INFO",
        "correlation_id": "1234567890",
        "message": f"Function returned: {result}"
    }))
```

**Example input / output:**

```text
Input:
- Function call with a condition check

Output:
[{"timestamp": "2023-10-05T12:34:56.789Z", "level": "INFO", "correlation_id": "1234567890", "message": "Calling some_function"},
{"timestamp": "2023-10-05T12:34:56.790Z", "level": "ERROR", "correlation_id": "1234567890", "message": "Condition not met: True"},
{"timestamp": "2023-10-05T12:34:56.791Z", "level": "INFO", "correlation_id": "1234567890", "message": "Function returned: 42"}]
```

**Data:**

```json
{
  "timestamp": "{{now}}",
  "level": "{{log_level}}",
  "correlation_id": "{{correlation_id}}",
  "message": "{{message}}"
}
```

**Flow:**

1. Identify Logging Points
2. Define a Structured Format
3. JSON Template
4. Integrate Copilot forAutomated Logging
5. Integrate Code Reviews
6. Ensure Consistency Across theService

