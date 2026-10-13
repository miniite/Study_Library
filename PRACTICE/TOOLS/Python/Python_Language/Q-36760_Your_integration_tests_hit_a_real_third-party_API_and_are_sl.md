---
type: "practice"
difficulty: "Intermediate"
---

# Your integration tests hit a real third-party API and are slow and flaky. Compare responses/VCR-style cassettes, a mock server, and dependency injection for isolating them.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Python Tail Latency](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with integration tests that interact with real third-party APIs, you often face challenges like slowness due to network latency and flakiness due to external factors beyond your control. To address these issues, you can use several strategies including response/VCR-style cassettes, a mock server, or dependency injection. Each approach has its own trade-offs and benefits.

#### Triaging the Question

The goal is to compare three different methods for isolating third-party API calls in integration tests:

Response/VCR-style cassettes: Recording actual responses from the API.
Mock servers: Simulating the behavior of the external service without making real API calls.
Dependency injection: Mocking dependencies at a higher level.

#### Assumptions
You are working with Python and have access to popular libraries for testing, such as pytest and unittest.
The third-party API is HTTP-based (e.g., REST).
Your application uses external services via HTTP requests.
The test environment has the necessary dependencies installed.

#### First Principles Solution
Response/VCR-style cassettes

Concept: VCR (VCR.py) allows you to record and replay HTTP interactions during tests. This approach captures real responses from the third-party API, making your tests deterministic and isolated.

Steps:

Install vcrpy library.
Create a cassette file that records requests and their corresponding responses.
In your test cases, use the cassette to automatically replace real HTTP calls with recorded ones.
PYTHON
from vcr_unittest import VCRUnittest

class TestIntegration(VCRUnittest):
    def setUp(self):
        super().setUp()
        self.vcr.use_cassette('test_integration')

    def test_third_party_api(self):
        response = requests.get('https://api.example.com/data')
        assert response.status_code == 200

#### Pros

Records actual responses from the API, ensuring that your tests reflect real-world scenarios.
Can be used to validate expected behavior against actual data.

#### Cons

External service downtime can break tests if not updated in the cassette.
Requires maintaining and updating cassette files as APIs evolve.
Slower due to recording real interactions.
Mock servers

Concept: A mock server simulates the external API’s responses without making real HTTP requests. This approach is useful for creating controlled environments during testing.

Steps:

Set up a local HTTP server that mimics the behavior of the third-party service.
Configure your application to use this mock server in test cases instead of hitting the real API.
PYTHON
import http.server

class MockServer(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/data':
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'{"key": "value"}')
        else:
            super().do_GET()

if __name__ == "__main__":
    http.server.test(HandlerClass=MockServer, port=8080)

#### Pros

Fast and deterministic since no real HTTP requests are made.
Easier to control the responses and edge cases during tests.

#### Cons

Requires setting up and managing a local server.
Less accurate compared to VCR as it does not capture actual responses from the API.
Mocking complex APIs can be more challenging.
Dependency injection

Concept: Dependency injection allows you to inject mock objects or stubs into your application components. This approach is useful for isolating dependencies and making tests more focused on specific units of code.

Steps:

Define interfaces for external services in your application.
Use a mocking framework like unittest.mock to replace these interfaces with mocks during testing.
PYTHON
from unittest.mock import MagicMock

class Service:
    def get_data(self):
        pass

def test_third_party_api():
    service = MagicMock()
    service.get_data.return_value = {"key": "value"}

    response = requests.get('https://api.example.com/data', services=service)
    assert response.status_code == 200

#### Pros

Highly flexible and easy to use with existing codebases.
Can mock complex interactions between components.

#### Cons

May require significant refactoring of your application’s dependencies.
Less realistic compared to VCR as it doesn’t record actual API responses.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, each approach has its strengths and weaknesses:

VCR-style cassettes are best for recording and validating real interactions but require maintaining cassette files.
Mock servers provide a fast and deterministic environment but may not accurately reflect all API behaviors.
Dependency injection offers flexibility by allowing you to mock dependencies at higher levels, but it requires refactoring your code.

#### Common mistakes
include:

Over-relying on VCR without updating cassettes when APIs change.
Not setting up the necessary infrastructure for mock servers and dependency injection.
Misconfiguring mocks in a way that breaks expected behavior during tests.

#### Follow-up
questions might include:

How do you handle rate limits or authentication with these approaches?
What are your thoughts on combining multiple techniques to achieve the best results?

**Comparison:**

```
Technique	Pros	Cons
Response/VCR	Records actual responses, accurate scenarios	External downtime can break tests, requires maintenance
Mock Server	Fast and deterministic, easy to control	Requires setting up server, less realistic API behavior
Dependency Injection	Flexible, easy to use with existing code	May require significant refactoring, less realistic API interactions
```

**Code:**

```

from vcr_unittest import VCRUnittest

class TestIntegration(VCRUnittest): def setUp(self): super().setUp() self.vcr.use_cassette('test_integration')

CODE
Copy
def test_third_party_api(self):
    response = requests.get('https://api.example.com/data')
    assert response.status_code == 200
CODE
Copy
```python
import http.server

class MockServer(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/data':
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'{"key": "value"}')
        else:
            super().do_GET()

if __name__ == "__main__":
    http.server.test(HandlerClass=MockServer, port=8080)

import unittest
from unittest.mock import MagicMock

class Service:
    def get_data(self):
        pass

def test_third_party_api():
    service = MagicMock()
    service.get_data.return_value = {"key": "value"}

    response = requests.get('https://api.example.com/data', services=service)
    assert response.status_code == 200
```

**Example input / output:**

```text
Input:
- Test case using VCR cassettes

Output:
- Deterministic test runs with no real API calls
```

**Data:**

```json
{
    "key": "value"
}
```

**Flow:**

1. Define API
2. Use VCR/Cassettes
3. Set Up Mock Server
4. Control Responses
5. Inject Mocks/DI
6. Isolate Dependencies
7. Test Scenarios

