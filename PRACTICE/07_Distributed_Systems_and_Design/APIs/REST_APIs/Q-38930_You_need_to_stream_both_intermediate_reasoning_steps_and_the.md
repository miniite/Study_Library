---
type: "practice"
difficulty: "Deep"
---

# You need to stream both intermediate reasoning steps and the final answer over one SSE connection so a UI can show the agent's progress. How do you design the event types and schema so a client can render steps, tool calls, and the final result distinctly?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Operations Behind an Endpoint](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Distributed_Operations_Behind_an_Endpoint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient and clear REST API that streams both intermediate reasoning steps and the final answer over one Server-Sent Events (SSE) connection, we need to carefully craft event types and a schema that allows clients to render these steps distinctly. This involves understanding the structure of each event type and ensuring it carries all necessary information without redundancy.

The key is to define specific event types for different stages—intermediate reasoning, tool calls, and the final result—and ensure they are easily distinguishable on the client side. By doing so, we can provide a seamless and informative user experience.

First, let’s break down what needs to be conveyed:

Intermediate Reasoning Steps: These steps represent the ongoing thought process of the AI agent.
Tool Calls: This includes interactions with external tools or APIs that are part of the reasoning process.
Final Result: The ultimate outcome produced by the agent.

Let’s start by defining these event types and their schemas:

**Code:**

```python
class EventTypes:
    REASONING_STEP = "reasoning_step"
    TOOL_CALL = "tool_call"
    FINAL_RESULT = "final_result"

class ReasoningStepSchema:
    def __init__(self, step_id: int, message: str):
        self.step_id = step_id
        self.message = message

class ToolCallSchema:
    def __init__(self, call_id: int, tool_name: str, arguments: dict):
        self.call_id = call_id
        self.tool_name = tool_name
        self.arguments = arguments

class FinalResultSchema:
    def __init__(self, result_value: str):
        self.result_value = result_value
```

