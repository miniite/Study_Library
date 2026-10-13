---
type: "practice"
difficulty: "Basic"
---

# How do integration tests differ from unit tests in what they need in the pipeline?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pre-merge vs Post-merge Failures](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Integration tests and unit tests serve different purposes within a CI/CD pipeline, but both are crucial components of ensuring software quality.

TL;DR:

Unit Tests: Focus on testing individual components or functions in isolation.
Integration Tests: Test the interaction between these components to ensure they work together as expected.

#### Triaging the Question

The question is asking you to differentiate between integration tests and unit tests. The interviewer might be probing your understanding of test types, their roles in a CI/CD pipeline, and how they contribute to ensuring software reliability.

#### First-Principles Solution
Unit Tests
Definition: Unit tests verify that individual units (functions, methods) behave as expected.
Scope: They operate on the smallest logical components of your codebase.
Dependencies: Typically mock or stub external dependencies to test in isolation.
Purpose: Ensure correctness and consistency at a granular level.
Integration Tests
Definition: Integration tests verify that different parts of an application interact correctly with each other.
Scope: They focus on the interaction between components, such as services, databases, or APIs.
Dependencies: Often require actual external systems to be functional.
Purpose: Ensure that when pieces come together, they still perform as expected.

#### Common Mistakes
and Follow-Ups
Confusion with Scope: Sometimes candidates mix up the scope of what unit tests and integration tests cover.
Overreliance on One Type: It’s important to balance both types; neither should be neglected.
Dependency Management: Managing dependencies correctly is crucial for both test types.

#### Closing Summary

Understanding the difference between unit tests and integration tests helps in designing a robust CI/CD pipeline. Both are necessary but serve different purposes, ensuring that your application works at various levels of complexity.

**Comparison:**

```
Test Type	Scope	Dependencies	Purpose
Unit	Individual units	Mocked	Ensure correctness of each part
Integration	Inter-component	Real or mocked	Verify interactions work as expected
```

**Code:**

```python
# Example of Unit Test (using Python unittest)
import unittest

class TestAddFunction(unittest.TestCase):
    def test_add(self):
        from my_module import add
        self.assertEqual(add(1, 2), 3)

# Example of Integration Test (using pytest with fixtures for mocking dependencies)
import pytest
from my_module import service

@pytest.fixture
def mock_db():
    # Mocking database interaction
    return "mocked data"

def test_integration(mock_db):
    result = service.get_data(mock_db)
    assert result == "expected outcome from integration"
```

**Example input / output:**

```

Unit Test Example:

PLAINTEXT
Copy
Input: test_add(1, 2)
Output: 3

Integration Test Example:

PLAINTEXT
Copy
Mock db = mocked data
Service.get_data(mock_db) -> expected outcome from integration
```

**Data:**

```json
{
  "unit_test_example": {
    "test_add": {
      "input": [1, 2],
      "output": 3
    }
  },
  "integration_test_example": {
    "mock_db": "mocked data",
    "service.get_data_output": "expected outcome from integration"
  }
}
```

**Flow:**

1. Codebase
2. Unit Tests
3. Unit Test Results
4. Integration Tests
5. Integration Test Results
6. Pipeline Deployment

