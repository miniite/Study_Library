---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle validation errors in a REST API, and what status code and body would you return?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling validation errors in a REST API, it's crucial to provide clear feedback about the error while adhering to best practices such as HTTP status codes and content formats. In this scenario, we need to define how to return appropriate status codes and error bodies that convey relevant information without exposing sensitive data.

The core idea is to use HTTP status codes effectively to signal the nature of the error, and include a structured JSON response body with detailed error messages and other necessary details such as fields that failed validation. This approach ensures that clients can understand what went wrong and how they can rectify it.

First, let's triage the question:

HTTP Status Codes: We need to choose an appropriate status code based on the type of error.
Error Response Body: The response should include a clear message, field-level errors if applicable, and other relevant details.
Step-by-Step Solution

We'll start with common validation errors such as missing fields or incorrect data types.

Identify the Error Type: Determine whether the issue is related to a specific field or more general.
Choose the Right Status Code: Use 400 Bad Request for client-side errors like invalid input, and 422 Unprocessable Entity for more complex validation issues.
Structure the Response Body: Return a JSON object with an error key and a nested details object if needed.

#### Example

Let's consider a user registration endpoint that requires email, password, and age fields.

Input Example:
JSON
{
  "email": "",
  "password": "pass123",
  "age": 18
}
Expected Response with Error Handling:

Empty Email (400 Bad Request):

JSON
{
  "error": "Validation failed",
  "details": [
    {
      "field": "email",
      "message": "Email cannot be empty"
    }
  ]
}

Invalid Age (422 Unprocessable Entity):

JSON
{
  "error": "Validation failed",
  "details": [
    {
      "field": "age",
      "message": "Age must be at least 18"
    }
  ]
}

#### Implementation
Steps
Define Validation Functions: Create validation functions to check the input fields.
Error Handling Logic: Implement logic in your API to handle these errors and format them as described above.
Sample Code
JAVASCRIPT

#### cons
t validateEmail = (email) => {
  if (!email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    return { field: "email", message: "Email cannot be empty or is invalid" };
  }
  return null;
};

#### cons
t validateAge = (age) => {
  if (!age || age < 18) {
    return { field: "age", message: "Age must be at least 18" };
  }
  return null;
};
Response Handling
JAVASCRIPT

#### cons
t user = req.body;

#### cons
t errors = [];

if (validateEmail(user.email)) errors.push(validateEmail(user.email));
if (validateAge(user.age)) errors.push(validateAge(user.age));

if (errors.length > 0) {

#### cons
t response = {
    error: "Validation failed",
    details: errors,
  };
  return res.status(400).json(response);
}

// Proceed with normal registration logic

#### Closing Summary
and Common Mistakes
Clear Error Messages: Always provide clear, actionable messages to help the client understand what went wrong.
HTTP Status Codes: Use appropriate status codes (400 vs 422) to indicate the nature of the error.
Field-Level Errors: Include field-level details for better debugging.

#### Common mistakes
include:

Returning generic "Internal Server Error" responses.
Not providing enough detail in error messages, making it hard for clients to fix issues.
Using inconsistent or non-standard status codes and response formats.

**Comparison:**

```
Field	Message
email	Email cannot be empty
```

**Code:**

```

const validateEmail = (email) => { if (!email || !/^[^\s@]+@[^\s@]+.[^\s@]+$/.test(email)) { return { field: "email", message: "Email cannot be empty or is invalid" }; } return null; };

const validateAge = (age) => { if (!age || age < 18) { return { field: "age", message: "Age must be at least 18" }; } return null; };

CODE
Copy

```javascript
const user = req.body;

const errors = [];

if (validateEmail(user.email)) errors.push(validateEmail(user.email));
if (validateAge(user.age)) errors.push(validateAge(user.age));

if (errors.length > 0) {
  const response = {
    error: "Validation failed",
    details: errors,
  };
  return res.status(400).json(response);
}
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "email": "",
  "password": "pass123",
  "age": 18
}

Output:

JSON
Copy
{
  "error": "Validation failed",
  "details": [
    {
      "field": "email",
      "message": "Email cannot be empty"
    }
  ]
}
```

**Data:**

```

{ "email": "", "password": "pass123", "age": 18 }

CODE
Copy

```json
{
  "error": "Validation failed",
  "details": [
    {
      "field": "email",
      "message": "Email cannot be empty"
    }
  ]
}
```

**Flow:**

1. No
2. No
3. Validate Email
4. Is Valid?
5. Add Error to List
6. Validate Age
7. Is Valid?
8. Add Error to List
9. Generate Response with Errors
10. Return JSON Response (400)

