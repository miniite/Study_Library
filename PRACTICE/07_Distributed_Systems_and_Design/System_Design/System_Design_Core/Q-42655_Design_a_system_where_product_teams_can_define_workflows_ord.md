---
type: "practice"
difficulty: "Expert"
---

# Design a system where product teams can define workflows (order fulfillment, KYC) as long-running orchestrations that survive process restarts and take days to complete. Would you build on a durable-execution engine or roll your own sagas, and why?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system where product teams can define workflows as long-running orchestrations that survive process restarts and take days to complete, we need to consider the core requirements of durability, fault tolerance, and manageability. The key decision points include whether to build on an existing durable execution engine or roll our own sagas.

Firstly, let's triage the question:

What is being asked: We need to decide between a pre-built solution (durable execution engine) and developing a custom saga-based system.
What are we probing for: The interviewer wants to understand your design thinking, trade-offs between off-the-shelf vs. custom solutions, and how you balance complexity with reliability.

#### Triage

We will analyze both options: using a durable execution engine like AWS Step Functions, and rolling our own saga-based system. Durable execution engines are well-established for handling long-running processes, while sagas provide more control over complex transactional workflows.

#### First-Principles Solution
Option 1: Durable Execution Engine (e.g., AWS Step Functions)

#### Pros

Ease of Use: No need to implement complex state management and retry logic.
Reliability: Built-in support for handling retries, timeouts, and fault tolerance.
Scalability: Managed service with automatic scaling capabilities.

#### Cons

Cost: Potentially higher costs if not optimized correctly.
Flexibility: Less customization compared to a custom solution.
Option 2: Roll Your Own Sagas

#### Pros

Customization: Full control over the workflow logic and state management.
Performance: Potential for better performance in complex, customized scenarios.

#### Cons

Complexity: Higher development effort required to implement robust state management, retries, and fault tolerance.
Operations: Increased operational overhead, including monitoring and error handling.
Step-by-Step Solution
Define the Workflow Structure: Break down each workflow into smaller, manageable tasks that can be individually executed. Use a domain-specific language (DSL) or YAML files to define these workflows.
Choose a Durable Execution Engine:
AWS Step Functions: Define states for each task and transitions between them using the AWS console or programming interfaces.
State Machine Languages: Utilize state machine languages like AWS Step Functions JSON format to define complex workflows with branches, loops, and error handling.
Implement State Management: Ensure that each step maintains its own state and can be resumed from where it left off after a failure.
Error Handling and Retries: Implement retry logic based on specific error types and use timeouts to manage long-running processes.
Monitoring and Logging: Set up detailed logging and monitoring to track the progress of workflows and detect issues early.

#### Closing Summary

In conclusion, using a durable execution engine like AWS Step Functions is generally more practical for most use cases due to its robust features and ease of use. However, if you need highly customized workflows with unique state management requirements, rolling your own saga-based system might be justified despite the added complexity.

#### Common Mistakes/Follow-Ups
Overlooking State Management: Failing to properly manage states can lead to data inconsistencies.
Ignoring Fault Tolerance: Not implementing proper retry and error handling mechanisms can cause workflow failures.
Underestimating Customization Needs: Assuming that a pre-built solution will meet all requirements without considering custom needs.
Final Thoughts

Choosing the right approach depends on the specific requirements, existing infrastructure, and team expertise. Always consider the trade-offs between ease of implementation and long-term maintainability.

**Comparison:**

```
Start
Task1
Task2
End
```

**Code:**

```python
# Example Step Function Definition (AWS)
def define_step_function(name):
    return {
        "Comment": "A short description",
        "StartAt": "Task1",
        "States": {
            "Task1": {
                "Type": "Task",
                "Resource": "arn:aws:lambda:us-east-1:<account-id>:function:task1",
                "Next": "Task2"
            },
            "Task2": {
                "Type": "Task",
                "Resource": "arn:aws:lambda:us-east-1:<account-id>:function:task2",
                "End": True
            }
        }
    }

# Example Custom Saga Workflow (Pseudocode)
class OrderFulfillmentSaga:
    def __init__(self, order_id):
        self.order_id = order_id

    def start(self):
        # Initialize states and dependencies
        pass

    def execute_task(task: Task):
        try:
            task.execute()
        except Exception as e:
            handle_exception(e)

# Example Error Handling (Pseudocode)
def handle_exception(error):
    if error.type == "Timeout":
        retry_later()
    elif error.type == "DataCorrupted":
        rollback_transactions()
```

**Example input / output:**

```text
Input:
{
  "Comment": "A short description",
  "StartAt": "Task1",
  "States": {
    "Task1": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:<account-id>:function:task1",
      "Next": "Task2"
    },
    "Task2": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:<account-id>:function:task2",
      "End": true
    }
  }
}

Output:
{
  "States": [
    {"Name": "Task1", "Type": "Task"},
    {"Name": "Task2", "Type": "Task"}
  ]
}
```

**Data:**

```json
{
  "States": {
    "Task1": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:<account-id>:function:task1",
      "Next": "Task2"
    },
    "Task2": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:<account-id>:function:task2",
      "End": true
    }
  }
}
```

**Flow:**

1. digraph Workflow {
2. node [shape=box]
3. Start -> Task1
4. Task1 -> Task2
5. Task2 -> End
6. }

