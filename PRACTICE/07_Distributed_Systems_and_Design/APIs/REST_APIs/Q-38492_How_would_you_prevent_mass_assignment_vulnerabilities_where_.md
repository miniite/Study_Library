---
type: "practice"
difficulty: "Deep"
---

# How would you prevent mass assignment vulnerabilities where a client sends extra fields like isAdmin in a request body?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address mass assignment vulnerabilities, we need to ensure that our backend application strictly controls which fields are allowed in incoming requests and that any unexpected or unauthorized fields are properly filtered out or ignored. This involves a combination of input validation and controlled model binding.

#### Triage
Understanding the Question

The question is essentially asking how to prevent clients from sending extra fields, such as isAdmin, which could potentially give them administrative privileges without proper authorization. The key here is to ensure that only predefined fields are allowed in the request body, and any additional fields should be rejected or ignored.

Key Assumptions:
We use a framework that supports model binding (e.g., Express.js with Joi).
The client sends JSON data.
There's an existing backend API using REST principles.
Step-by-Step Solution
1. Define the Allowed Fields

First, we need to clearly define which fields are allowed for each endpoint. This can be done by creating a schema or a set of rules that govern what is accepted and what is not.

2. Implement Input Validation

Use input validation libraries like Joi in Node.js to ensure that only the defined fields are processed. Here’s an example using Express.js and Joi:

JAVASCRIPT

#### cons
t express = require('express');

#### cons
t Joi = require('@hapi/joi');

#### cons
t app = express();
app.use(express.json());

// Define allowed fields for a user registration endpoint

#### cons
t schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(6).max(30).required(),
  // Other valid fields...
});

app.post('/register', (req, res) => {

#### cons
t { error } = schema.validate(req.body);
  if (error) {
    return res.status(400).send(error.details[0].message);
  }
  // Proceed with registration logic
});
3. Use Model Binding Wisely

If your framework supports model binding, make sure to use it only for fields that are explicitly allowed. For example:

JAVASCRIPT

#### cons
t userSchema = {
  email: String,
  password: String,
  // Other valid fields...
};

app.post('/register', (req, res) => {

#### cons
t user = req.body;
  if (!userSchema.validate(user).error) {
    // Proceed with registration logic
  } else {
    return res.status(400).send('Invalid input');
  }
});
4. Filter Extra Fields

If the framework doesn't support strict validation, you can manually filter out extra fields:

JAVASCRIPT
app.post('/register', (req, res) => {

#### cons
t allowedFields = ['email', 'password'];

#### cons
t filteredBody = Object.keys(req.body).reduce((acc, key) => {
    if (allowedFields.includes(key)) {
      acc[key] = req.body[key];
    }
    return acc;
  }, {});

  // Proceed with registration logic using `filteredBody`
});
5. Handling Unexpected Fields

Handle unexpected fields by rejecting them or logging an error, which can help in auditing and debugging:

JAVASCRIPT
app.post('/register', (req, res) => {

#### cons
t allowedFields = ['email', 'password'];
  for (let key of Object.keys(req.body)) {
    if (!allowedFields.includes(key)) {
      return res.status(400).send('Unexpected field: ' + key);
    }
  }

  // Proceed with registration logic
});

#### Closing Summary
and Common Mistakes

In summary, the primary approach is to define a schema of allowed fields and use it to validate incoming requests. By doing so, we ensure that only expected data is processed, mitigating mass assignment vulnerabilities.

#### Common Mistakes

Not Defining Allowed Fields: Failing to specify which fields are permitted.
Overly Permissive Schemas: Allowing too many fields, making it difficult to validate and secure the API.
Manual Filtering Flaws: Manually filtering extra fields can lead to oversight or errors if not implemented correctly.

By following these steps and avoiding common pitfalls, we can effectively prevent mass assignment vulnerabilities in our REST APIs.

