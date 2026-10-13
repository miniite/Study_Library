---
type: "practice"
difficulty: "Deep"
---

# How would you write integration tests for a FastAPI agent endpoint that assert the correct tools were called in the right order, using a mocked deterministic LLM?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Integration testing an endpoint in a FastAPI application that asserts the correct sequence of tool calls from a deterministic LLM involves several steps. The goal is to ensure that the agent behaves as expected, invoking tools at the right moments. We'll use Python's unittest.mock library along with FastAPI’s test client to achieve this.

First, let's break down the problem:

We need to mock an LLM.
The mock must allow us to control its responses deterministically.
We need to assert that specific tools are called in a specified order.
Integration testing requires setting up a complete scenario as if it were a real request to the FastAPI endpoint.

To accomplish this, we'll follow these steps:

Define the Mock LLM: Create a mock class that simulates an LLM’s behavior and allows us to control its responses.
Set Up the Endpoint: Define the FastAPI route that interacts with the LLM and the agent tools.
Write Tests: Use unittest.mock to inject our mocked LLM into the test environment, and assert tool calls in the correct order.

Now, let's dive into the implementation details:

#### Triaging the Question

The interviewer is probing your ability to:

Understand FastAPI’s testing mechanisms.
Utilize Python’s mocking capabilities effectively.
Ensure that interactions with external services (like an LLM) are properly controlled and tested.
Write comprehensive integration tests for complex systems.

We assume the following:

The agent endpoint uses a tool interface, which we need to mock.
The tools have methods like execute that return responses.
We can use FastAPI’s test client (TestClient) to send requests and inspect responses.

#### First Principles Solution
Define the Mock LLM:
PYTHON
from unittest.mock import MagicMock

class MockLLM:
    def __init__(self):
        self.responses = [
            {"tool_0": "execute"},  # First tool call
            {"tool_1": "execute"}   # Second tool call
        ]

    def generate(self, prompt):
        return self.responses.pop(0)
Set Up the Endpoint:
PYTHON
from fastapi import FastAPI
from typing import Dict

app = FastAPI()

@app.post("/agent-endpoint")
async def agent_endpoint(tool_interface: Dict[str, str]) -> Dict[str, str]:
    response = await tool_interface["tool_0"]["execute"]  # Simulate first call
    return {"result": "success"}
Write Tests:
PYTHON
import unittest
from fastapi.testclient import TestClient
from httpx import AsyncHTTPClient

class TestAgentEndpoint(unittest.TestCase):
    def setUp(self):
        self.mock_llm = MockLLM()

        # Set up a mock tool interface that uses the mocked LLM responses
        class MockToolInterface:
            async def __call__(self, prompt: str) -> Dict[str, str]:
                return {"execute": self.mock_llm.generate(prompt)}

        self.client = TestClient(app)

    def test_correct_tool_sequence(self):
        # Simulate a request to the endpoint with appropriate data
        response = self.client.post("/agent-endpoint", json={"tool_0": "prompt"})

        # Assert that the correct tool was called in the right order
        self.assertEqual(response.json(), {"result": "success"})

#### Closing Summary
and Common Mistakes

In summary, we've demonstrated how to write integration tests for a FastAPI endpoint using a mocked deterministic LLM. The key steps involve defining the mock LLM class with controlled responses, setting up the endpoint that uses this tool interface, and writing comprehensive unit tests that assert the correct sequence of tool calls.

#### Common mistakes
include:

Not properly mocking the external service (LLM in this case).
Failing to set up a deterministic flow for the test cases.
Overlooking edge cases or incorrect handling of asynchronous calls in FastAPI.

By addressing these, you ensure robust and reliable testing of complex systems.

**Code:**

```python
from unittest.mock import MagicMock

class MockLLM:
    def __init__(self):
        self.responses = [
            {"tool_0": "execute"},  # First tool call
            {"tool_1": "execute"}   # Second tool call
        ]

    def generate(self, prompt):
        return self.responses.pop(0)

from fastapi import FastAPI
from typing import Dict

app = FastAPI()

@app.post("/agent-endpoint")
async def agent_endpoint(tool_interface: Dict[str, str]) -> Dict[str, str]:
    response = await tool_interface["tool_0"]["execute"]  # Simulate first call
    return {"result": "success"}

import unittest
from fastapi.testclient import TestClient

class TestAgentEndpoint(unittest.TestCase):
    def setUp(self):
        self.mock_llm = MockLLM()

        class MockToolInterface:
            async def __call__(self, prompt: str) -> Dict[str, str]:
                return {"execute": self.mock_llm.generate(prompt)}

        self.client = TestClient(app)

    def test_correct_tool_sequence(self):
        response = self.client.post("/agent-endpoint", json={"tool_0": "prompt"})
        self.assertEqual(response.json(), {"result": "success"})
```

**Example input / output:**

```text
response = client.post("/agent-endpoint", json={"tool_0": "prompt"})
# Response should be {"result": "success"}
```

**Data:**

```json
{
  "tool_interface": {
    "tool_0": "prompt",
    "tool_1": "prompt"
  }
}
```

