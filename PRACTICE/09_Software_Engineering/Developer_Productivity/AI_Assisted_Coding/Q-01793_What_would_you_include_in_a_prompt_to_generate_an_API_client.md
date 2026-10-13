---
type: "practice"
difficulty: "Intermediate"
---

# What would you include in a prompt to generate an API client matching an OpenAPI spec?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompting for Validation and Explanations](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Validation_and_Explanations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When generating an API client from an OpenAPI specification, your prompt should be clear and detailed enough to guide the AI assistant effectively. Here’s how you can structure a high-quality prompt:

TL;DR

A well-crafted prompt for generating an API client includes specifying the context (e.g., programming language), desired features (like authentication methods or error handling), and any custom configurations required.

#### Triaging the Question

The interviewer is likely probing your understanding of OpenAPI specs, how to effectively communicate requirements to an AI tool like Copilot/Cursor, and your ability to handle practical coding tasks. Key assumptions include familiarity with common API client frameworks (e.g., axios for JavaScript) and knowledge of different authentication methods.

#### First-Principles Solution
Context Setup: Start by setting the context: programming language, framework or library you’re using.
OpenAPI Spec: Include a snippet of the OpenAPI specification to ensure the AI understands the API structure.
Functional Requirements: Detail what functionalities you need in your client (e.g., authentication, error handling).
Custom Configurations: Specify any custom configurations required for the generated code.

#### Example
Prompt
PLAINTEXT
I'm working on a JavaScript project and need to generate an API client from the following OpenAPI specification:

```yaml
openapi: 3.0.0
info:
  title: Sample API
  version: "1.0"
paths:
  /users:
    get:
      summary: Get users list
      responses:
        '200':
          description: A successful response
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

I need the client to use the axios library for making HTTP requests and include functionality for token-based authentication with a custom error handler.

CODE

### Closing Summary + Common Mistakes/Follow-Ups
- **Mistake**: Failing to provide enough context or including too much extraneous information, which can confuse the AI.
- **Follow-Up**: The interviewer might ask about specific authentication methods or how you would handle edge cases like rate limiting.

```plaintext
By providing a clear and concise prompt that includes both the OpenAPI spec and your functional requirements, you can effectively guide Copilot/Cursor to generate an API client tailored to your needs. Always test the generated code thoroughly to ensure it meets all your requirements.

#### Common Mistakes
Overly Complex Prompts: Providing too much detail or irrelevant information can confuse the AI.
Ignoring Context: Not specifying the programming language, framework, or library can lead to incorrect implementations.

**Comparison:**

```
Prompt Elements	Description
Context	Programming language, framework, library
OpenAPI Spec	API structure details
Functional Req.	Authentication methods, error handling
Custom Config	Specific configurations
```

**Code:**

```javascript
// Example of using axios for API client in JavaScript
const axios = require('axios');

async function getUserList() {
  try {
    const response = await axios.get('/users', { headers: { Authorization: `Bearer ${token}` } });
    console.log(response.data);
  } catch (error) {
    if (!axios.isAxiosError(error)) throw error;
    console.error(`HTTP Status Code: ${error.response.status}`);
    console.error(`Response Data:`, error.response.data);
  }
}
```

**Example input / output:**

```
Input: The provided OpenAPI spec and the functional requirements for the client.
Output: Generated API client code in JavaScript that includes token-based authentication with a custom error handler.
PLAINTEXT
Copy
{
  "clientCode": "// Code generated from the given prompt and OpenAPI spec"
}
```

**Data:**

```yaml
openapi: 3.0.0
info:
  title: Sample API
  version: "1.0"
paths:
  /users:
    get:
      summary: Get users list
      responses:
        '200':
          description: A successful response
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

**Flow:**

1. Define Context
2. Provide OpenAPI Spec
3. Detail Functional Requirements
4. Specify Custom Configurations

