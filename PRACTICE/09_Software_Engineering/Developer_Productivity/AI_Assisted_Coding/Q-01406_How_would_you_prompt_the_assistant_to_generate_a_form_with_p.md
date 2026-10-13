---
type: "practice"
difficulty: "Intermediate"
---

# How would you prompt the assistant to generate a form with proper client-side validation AND explain which rules must be re-enforced on the server?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompting for Validation and Explanations](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Validation_and_Explanations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with AI-assisted coding tools like GitHub Copilot, generating a form with both client-side and server-side validation is essential to ensure data integrity and user experience. The key idea here is to craft prompts that guide the tool to produce code that meets specific validation requirements.

In this case, you would want the assistant to create an HTML form, integrate client-side JavaScript for initial validation checks, and outline server-side validation rules in a clear way. Here’s how we can break down the problem from first principles:

Identify the Key Requirements:

Generate a simple HTML form with input fields.
Ensure client-side validation to provide real-time feedback to the user.
Outline necessary backend validation logic.

#### Triage
and Assumptions:

We assume the use of modern web technologies such as HTML, JavaScript (React or vanilla JS), and Node.js on the server side.
The form will contain at least a text input for name and an email field.
The client-side validation should check for non-empty fields and valid email format.
Server-side validation must enforce these rules strictly to prevent any malicious inputs.

Step-by-Step Solution:

Step 1: Define the HTML Form

Start by defining a simple HTML form:

HTML
<form id="userForm">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name"><br><br>

    <label for="email">Email:</label>
    <input type="email" id="email" name="email"><br><br>

    <button type="submit">Submit</button>
</form>
Step 2: Implement Client-Side Validation

Next, use JavaScript to add real-time validation:

JAVASCRIPT
document.getElementById('userForm').addEventListener('submit', function(event) {
    event.preventDefault(); // Prevent the default form submission

#### cons
t name = document.getElementById('name');

#### cons
t email = document.getElementById('email');

    if (!name.value.trim() || !email.value.trim()) {
        alert('All fields are required!');
        return;
    }

#### cons
t validEmailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!validEmailPattern.test(email.value)) {
        alert('Invalid email format!');
        return;
    }

#### cons
ole.log('Form data is valid and can be submitted.');
});
Step 3: Outline Server-Side Validation

Finally, outline the server-side validation logic. This could be in Node.js using an Express app:

JAVASCRIPT

#### cons
t express = require('express');

#### cons
t bodyParser = require('body-parser');

#### cons
t app = express();

app.use(bodyParser.json());

// Server-side validation function
function validateFormData(data) {
    if (!data.name.trim() || !data.email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
        return false;
    }
    return true;
}

app.post('/submit', (req, res) => {

#### cons
t formData = req.body;

    if (validateFormData(formData)) {
        // Process and save the data

#### cons
ole.log('Valid form data received:', formData);
        res.send('Form data processed successfully.');
    } else {
        // Handle invalid data
        res.status(400).send('Invalid form data');
    }
});

app.listen(3000, () => console.log('Server running on port 3000'));

#### Closing Summary
and Common Mistakes:
Ensure all fields have non-empty values.
Use regular expressions to validate emails accurately.
Always sanitize inputs before processing them.

#### Common Mistakes

Overlooking server-side validation, assuming client-side is enough.
Failing to handle edge cases like multiple spaces or special characters in email validation.
Not logging and handling errors properly on both client and server sides.

By following these steps and addressing common pitfalls, you can create a robust form with proper client-side and server-side validation using AI-assisted coding tools like Copilot.

**Code:**

```
<form id="userForm"> <label for="name">Name:</label> <input type="text" id="name" name="name"><br><br>
CODE
Copy
<label for="email">Email:</label>
<input type="email" id="email" name="email"><br><br>

<button type="submit">Submit</button>
</form> ```
JAVASCRIPT
Copy
document.getElementById('userForm').addEventListener('submit', function(event) {
    event.preventDefault(); // Prevent the default form submission

    const name = document.getElementById('name');
    const email = document.getElementById('email');

    if (!name.value.trim() || !email.value.trim()) {
        alert('All fields are required!');
        return;
    }

    const validEmailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!validEmailPattern.test(email.value)) {
        alert('Invalid email format!');
        return;
    }

    console.log('Form data is valid and can be submitted.');
});
JAVASCRIPT
Copy
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());

// Server-side validation function
function validateFormData(data) {
    if (!data.name.trim() || !data.email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
        return false;
    }
    return true;
}

app.post('/submit', (req, res) => {
    const formData = req.body;

    if (validateFormData(formData)) {
        // Process and save the data
        console.log('Valid form data received:', formData);
        res.send('Form data processed successfully.');
    } else {
        // Handle invalid data
        res.status(400).send('Invalid form data');
    }
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

