---
type: "practice"
difficulty: "Intermediate"
---

# How would you use pytest fixtures to inject a fake LLM client and a fake vector store into your Flask test client?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Streaming Responses](<../../../../TOOLS/Python/Flask/Streaming_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively integrate a fake LLM (Language Model) client and a fake vector store with pytest fixtures in a Flask application, we first need to understand that fixtures allow us to create setup code that can be reused across multiple tests. This is particularly useful for mocking external dependencies like an LLM or vector store during our test runs.

#### Triage
What the Question Is Asking

The question is asking how to use pytest fixtures to mock out a fake LLM client and a fake vector store, which are often used as dependencies in real-world applications. The goal is to ensure that tests can run independently of external services while still providing realistic scenarios for testing our Flask application.

#### First-Principles Solution
Define the Fixtures: Create fixtures for both the LLM client and the vector store.
Use the Fixtures in Tests: Inject these mocks into your test functions to simulate real-world interactions without needing actual services.
Set Up Test Client: Use Flask’s test_client to interact with your application as if it were a real user.

#### Example
Implementation

First, let's define the fixtures using pytest:

PYTHON
import pytest
from flask import Flask
from mock_llm import FakeLLM  # Assume this is a custom LLM client
from mock_vector_store import FakeVectorStore  # Assume this is a custom vector store

@pytest.fixture
def fake_llm():
    return FakeLLM()

@pytest.fixture
def fake_vector_store():
    return FakeVectorStore()

Next, we use these fixtures in our test functions:

PYTHON
def test_example(test_client, fake_llm, fake_vector_store):
    # Assuming you have a route that interacts with the LLM and vector store
    response = test_client.post('/example', json={'text': 'Hello'})

    assert b'Greeting received' in response.data  # Example assertion

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using fixtures, we can create isolated and repeatable tests that do not rely on external services. This approach ensures our application behaves correctly under a wide range of scenarios.

#### Common Mistakes

Overcomplicating the Fixtures: Sometimes developers might overthink or overengineer their fixtures, making them unnecessarily complex.
Not Using test_client Properly: Failing to use Flask’s test_client can lead to issues where tests do not properly simulate a user interaction.

#### Follow-Ups

How would you handle different configurations of the LLM and vector store in your fixtures?
What are some best practices for organizing these fixtures across multiple test files?

**Example input / output:**

```text
def test_example(test_client, fake_llm, fake_vector_store):
    response = test_client.post('/example', json={'text': 'World'})

    assert b'Hello World' in response.data
```

