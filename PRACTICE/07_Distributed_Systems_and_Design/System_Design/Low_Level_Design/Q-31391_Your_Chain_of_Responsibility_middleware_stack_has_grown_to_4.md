---
type: "practice"
difficulty: "Deep"
---

# Your Chain of Responsibility middleware stack has grown to 40 handlers and ordering bugs are common; how do you make ordering explicit and testable, and how do you let one handler short-circuit safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issues in a growing Chain of Responsibility (CoR) middleware stack, we need to ensure that the order of handlers is explicit and testable while allowing safe short-circuiting. We'll achieve this through a structured approach that enforces ordered execution and provides clear testing mechanisms.

Triaging the Problem

The CoR pattern involves a chain of objects that each have the opportunity to handle a request. However, as the number of handlers grows (in this case, 40), it becomes increasingly difficult to manage their order correctly. This can lead to ordering bugs, making the system harder to reason about and test.

Core Solution

Our approach will involve structuring the CoR into a clear hierarchy and using dependency injection for handler dependencies. We'll leverage a priority-based mechanism to ensure ordered execution and provide testing interfaces to validate behavior under different scenarios.

Step-by-Step Solution

Define Handler Interfaces:

Each handler should implement a common interface, ensuring consistency in handling requests.
PYTHON
from abc import ABC, abstractmethod

class Handler(ABC):
    @abstractmethod
    def handle_request(self, request):
        pass

Create a Priority Queue for Handlers:

Use a priority queue to enforce the order of execution based on predefined priorities.
PYTHON
from heapq import heappush, heappop

class HandlerPriorityQueue:
    def __init__(self):
        self.queue = []

    def add_handler(self, handler):
        heappush(self.queue, (handler.priority, handler))

    def get_next_handler(self):
        return heappop(self.queue)[1]

Define Handlers with Priorities:

Each handler should have a priority attribute to determine its position in the queue.
PYTHON
class LoggingHandler(Handler):
    def __init__(self, next_handler=None):
        self.next_handler = next_handler
        self.priority = 10

    def handle_request(self, request):
        print(f"Logging: {request}")
        return self.next_handler.handle_request(request) if self.next_handler else None

Short-Circuit Mechanism:

Introduce a short_circuit flag to allow handlers to skip further processing.
PYTHON
class ShortCircuitHandler(Handler):
    def __init__(self, next_handler=None):
        self.next_handler = next_handler
        self.priority = 20

    def handle_request(self, request):
        if some_condition:
            print("Short-circuiting at short_circuit handler")
            return None
        return self.next_handler.handle_request(request) if self.next_handler else None

Integration and Testing:

Write unit tests to ensure the order of execution is correct and that handlers behave as expected.
PYTHON
def test_handler_order():
    queue = HandlerPriorityQueue()
    logging_handler = LoggingHandler(next_handler=ShortCircuitHandler())
    short_circuit_handler = ShortCircuitHandler(next_handler=None)
    queue.add_handler(short_circuit_handler)
    queue.add_handler(logging_handler)

    assert queue.get_next_handler().priority == 20
    assert queue.get_next_handler().priority == 10

test_handler_order()

#### Closing Summary
and Common Mistakes/Follow-Ups

This structured approach ensures that the CoR middleware stack remains manageable, even as it grows. By using a priority-based mechanism, we enforce explicit order of execution while providing flexibility through short-circuiting capabilities.

#### Common Mistakes

Ignoring the ordering of handlers can lead to subtle bugs.
Overlooking the need for clear testing interfaces can result in undetected issues.
Failing to use short-circuiting correctly might result in unnecessary processing overhead.

For follow-up, we could discuss additional optimizations such as caching or parallel execution strategies if needed. Additionally, we could explore how this approach scales with more complex scenarios and potential trade-offs between simplicity and performance.

**Code:**

```python
from abc import ABC, abstractmethod

class Handler(ABC):
    @abstractmethod
    def handle_request(self, request):
        pass

class HandlerPriorityQueue:
    def __init__(self):
        self.queue = []

    def add_handler(self, handler):
        heappush(self.queue, (handler.priority, handler))

    def get_next_handler(self):
        return heappop(self.queue)[1]

class LoggingHandler(Handler):
    def __init__(self, next_handler=None):
        self.next_handler = next_handler
        self.priority = 10

    def handle_request(self, request):
        print(f"Logging: {request}")
        return self.next_handler.handle_request(request) if self.next_handler else None

class ShortCircuitHandler(Handler):
    def __init__(self, next_handler=None):
        self.next_handler = next_handler
        self.priority = 20

    def handle_request(self, request):
        if some_condition:
            print("Short-circuiting at short_circuit handler")
            return None
        return self.next_handler.handle_request(request) if self.next_handler else None
```

