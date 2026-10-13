---
type: "practice"
difficulty: "Intermediate"
---

# How would you get Copilot to generate an accessible form (labels, ARIA) rather than just a functional one?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompting for Validation and Explanations](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Validation_and_Explanations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using AI-assisted coding tools like Copilot or similar systems, generating accessible forms is crucial for ensuring that all users can interact with web applications effectively. This involves not only creating functional form elements but also adding proper accessibility attributes such as labels and ARIA roles. To ensure that Copilot generates an accessible form, we need to provide clear instructions and guide the AI in several steps.

Firstly, let's understand what we're aiming for: a form with properly labeled input fields (e.g., text areas, checkboxes), appropriate aria-labels or aria-labelledby, and ensuring that all interactive elements have sufficient contrast ratios. The key is to instruct Copilot to generate these attributes explicitly.

#### Triaging the Question

The question is essentially asking how we can guide an AI tool like Copilot to prioritize accessibility when generating code for a form. This involves understanding the input instructions, knowing the necessary HTML and ARIA tags, and ensuring that the generated code follows best practices.

Key Assumptions
You have access to an environment where Copilot is enabled.
The developer has basic knowledge of web accessibility standards (WCAG 2.1).
The form's structure and purpose are clearly defined before using the AI tool.

#### Solution
from First Principles

To ensure that Copilot generates accessible forms, follow these steps:

Define Form Structure: Clearly define what elements will be in your form—input fields, labels, buttons, etc.
Explicitly Instruct on Accessibility: Use specific language to guide Copilot towards generating the necessary accessibility attributes.
Provide Contextual Information: Give Copilot context about the form's purpose and user group.
Step-by-Step Approach

Define Form Elements:

HTML
<form action="/submit" method="post">
  <!-- Input fields -->
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" aria-label="Full name required">

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" aria-describedby="email-help">

  <!-- Textarea -->
  <label for="message">Message:</label>
  <textarea id="message" name="message" aria-label="Your message"></textarea>

  <!-- Buttons -->
  <button type="submit" id="submit-btn" aria-label="Submit form">
    Submit
  </button>
</form>

Guide Copilot with Instructions:

Use comments or additional instructions in the code to guide Copilot.
For example, you can add a comment above each element specifying its purpose and required accessibility attributes.

Provide Contextual Help:

Explain the context of the form (e.g., it is for user registration, contact information).
Highlight any specific requirements or constraints that need to be met.

Review Generated Code: After Copilot generates the code, review it manually to ensure all required accessibility attributes are present and correctly applied.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Generating accessible forms with AI tools like Copilot is about providing clear instructions and ensuring that the tool understands the importance of accessibility. Common mistakes include:

Overlooking critical ARIA roles or labels.
Not reviewing the generated code thoroughly.
Failing to provide context about the form’s purpose.

#### Follow-ups
could involve discussing how to handle more complex forms, integrating with screen readers, and considering additional accessibility guidelines like WCAG 2.1.

**Code:**

```
HTML
Copy
<form action="/submit" method="post">
  <!-- Input fields -->
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" aria-label="Full name required">

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" aria-describedby="email-help">

  <!-- Textarea -->
  <label for="message">Message:</label>
  <textarea id="message" name="message" aria-label="Your message"></textarea>

  <!-- Buttons -->
  <button type="submit" id="submit-btn" aria-label="Submit form">
    Submit
  </button>
</form>
```

