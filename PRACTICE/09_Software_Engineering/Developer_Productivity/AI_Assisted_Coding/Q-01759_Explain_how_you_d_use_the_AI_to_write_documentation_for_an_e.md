---
type: "practice"
difficulty: "Basic"
---

# Explain how you'd use the AI to write documentation for an existing module you didn't author.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trust Boundaries for AI Generated Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Trust_Boundaries_for_AI_Generated_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how I would leverage AI-assisted coding, specifically tools like Copilot or similar, to write documentation for an existing module that I did not author, let's break down the process step by step.

Firstly, the core idea is to use AI as a supportive tool rather than a replacement. The AI can help us understand and document complex code more efficiently and accurately. By providing context, generating explanations, and suggesting comments, AI assists in creating high-quality documentation that complements human understanding and expertise.

#### Triaging the Question

The question asks about using AI to write documentation for an existing module. This involves several key aspects:

Understanding the Module: Since I didn't author it, I need to quickly grasp its purpose and functionality.
Generating Documentation Content: Using AI tools to generate appropriate documentation based on the code.
Reviewing and Refining: Ensuring that the generated content is clear, accurate, and useful for other developers or end-users.

#### Assumptions
The module has existing code but no current documentation.
The AI tool is designed to understand programming languages and can generate comments and explanations based on the code structure.
I have access to a development environment where I can run and test the AI-assisted coding tool.

#### First-Principles Solution

Quickly Familiarize with the Module:

Spend 15-20 minutes reviewing the module's files, focusing on key functions and variables.
Use IDE features like code navigation to understand the structure and dependencies.

Enable AI Assist:

Open my preferred editor or IDE that supports AI-assisted coding (e.g., VS Code with Copilot).
Enable the AI tool in a clean workspace, ensuring it’s properly set up for the project context.

Generate Documentation:

Use the AI to insert documentation blocks at relevant points in the code.
The AI can suggest comments explaining what each function does, variable usage, and edge cases.
For complex logic or algorithms, ask the AI to explain the steps involved.

Review and Refine:

Read through the generated documentation to ensure it accurately reflects the code's functionality.
Make necessary edits to improve clarity and readability.
Add any additional context that might help other developers understand the module better.

Iterate if Necessary:

If the AI-generated content is too generic or not detailed enough, prompt the AI with more specific questions or examples.
Continue refining until you are satisfied with the documentation quality.

#### Closing Summary
and Common Mistakes

By leveraging AI to write initial documentation, we can save time and ensure that the basic structure of the document is in place. However, it’s crucial to review and refine the content to make sure it is clear and useful. Common mistakes include relying too heavily on the AI without critical evaluation or missing out on key details that the AI might overlook.

Common Follow-Ups
How do you handle complex logic or algorithms when writing documentation?
What steps would you take if the AI-generated comments are not accurate enough?
How do you ensure that the generated documentation is consistent with existing coding standards and style guides?

**Comparison:**

| Function Name | Parameters | Returns |
|---------------|------------|---------|
| calculateArea  | length, width | area    |
| convertTemperature | temperature, fromUnit, toUnit | convertedTemp |

**Code:**

```javascript
// Example of using AI to generate a function comment in JavaScript
function calculateArea(length, width) {
  // AI-assisted documentation:
  // This function calculates the area of a rectangle.
  // It takes two parameters: length (number) and width (number).
  // Returns the area as a number.
  return length * width;
}
```

**Example input / output:**

```text
Input (AI-generated comment for calculateArea function):
This function calculates the area of a rectangle.
It takes two parameters: length (number) and width (number).
Returns the area as a number.

Output (Refined documentation):
/**
 * This function calculates the area of a rectangle.
 *
 * @param {number} length - The length of the rectangle.
 * @param {number} width - The width of the rectangle.
 * @returns {number} The calculated area.
 */
function calculateArea(length, width) {
  return length * width;
}
```

**Data:**

```json
{
  "module_name": "MathUtils",
  "functions": [
    {
      "name": "calculateArea",
      "params": ["length", "width"],
      "returns": "area"
    },
    {
      "name": "convertTemperature",
      "params": ["temperature", "fromUnit", "toUnit"],
      "returns": "convertedTemp"
    }
  ]
}
```

**Flow:**

1. Open IDE and Enable AI
2. Familiarize with the Module
3. Enable AI Assist
4. Generate Documentation
5. Review and Refine
6. Iterate if Necessary
7. Document is Ready

