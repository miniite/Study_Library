---
type: "practice"
difficulty: "Deep"
---

# How do you test that a FastAPI model service handles the full range of edge-case inputs (nulls, extreme values, unseen categories) gracefully rather than throwing 500s in production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your FastAPI model service gracefully handles a wide range of edge-case inputs—such as null values, extreme values, and unseen categories—we need to focus on comprehensive testing strategies. This involves both unit tests and integration tests, along with setting up robust validation mechanisms in the API itself. The goal is to prevent unexpected 500 errors in production by ensuring that any invalid or out-of-range inputs are handled gracefully.

Firstly, let's break down what the question is really asking: we need to validate that our FastAPI service can process a variety of edge cases without crashing or producing unhelpful error messages. This requires us to consider how different types of input might be processed and ensure that all potential failure points are covered in our tests.

**Comparison:**

```
Test Case	Input	Expected Status Code	Expected Response Body
Null	null	422	{"detail": ["Input cannot be null"]}
Extreme	"99999999999"	422	{"detail": ["Input is not a valid number"]}
Unseen	"unknown_cat"	400	{"detail": "Invalid input"}
```

**Example input / output:**

```text
# Example input->output for edge cases
{
    "null_input": {
        "request_body": {"input": null},
        "expected_status_code": 422,
        "expected_response_body": {"detail": ["Input cannot be null"]}
    },
    "extreme_value": {
        "request_body": {"input": "9999999999999999999999"},
        "expected_status_code": 422,
        "expected_response_body": {"detail": ["Input is not a valid number"]}
    },
    "unseen_category": {
        "request_body": {"input": "unknown_category"},
        "expected_status_code": 400,
        "expected_response_body": {"detail": "Invalid input"}
    }
}
```

**Flow:**

1. Yes
2. No
3. Validation Error
4. Unseen Category
5. Extreme Value
6. Start
7. Is Input Valid?
8. Return Prediction
9. Handle Error Type?
10. Return 422 UnprocessableEntity
11. Return 400 Bad Request
12. Return 422 UnprocessableEntity (with detail)
13. End

