---
type: "practice"
difficulty: "Intermediate"
---

# What's your strategy for giving the AI just enough context without pasting the whole repo?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Handling AI Inconsistency](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Handling_AI_Inconsistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When integrating an AI-assisted coding tool like Copilot into a development environment, one of the key challenges is providing the AI with sufficient context to offer relevant suggestions without overwhelming it or exposing too much code. The strategy should balance between giving enough information for meaningful assistance and maintaining security and privacy.

The question here is essentially about how to provide contextual snippets that help the AI understand the current context while minimizing the risk of unintentionally revealing sensitive parts of a project. Here’s a step-by-step approach based on first principles:

Identify Relevant Context: The AI needs to see relevant code files or sections where it can make sense of the current problem domain and provide accurate suggestions.
Filter Sensitive Information: Ensure that personal, private, or sensitive data is not exposed.
Dynamic Contextualization: Provide context dynamically based on the current editor cursor position and the surrounding code structure.

Let’s break this down further:

Identify Relevant Context:

File Level: The AI should see a few lines of the file currently in focus. For example, if you are working on utils.js, show lines around where the cursor is.
Project Structure: Include snippets from related files or folders that provide relevant context but avoid large sections.

Filter Sensitive Information:

Personal Data: Ensure no personal data (like user IDs, passwords) is visible to Copilot.
Sensitive Files: Avoid showing files containing sensitive information like database credentials, API keys, or internal project documentation.

Dynamic Contextualization:

Code Block Size: Use a fixed number of lines around the cursor. For instance, show 10 lines before and 5 lines after the current line.
Cursor Position: Update the context dynamically as you move the cursor within the file to ensure Copilot always has the most relevant information.

Here’s an example strategy:

File Context:
PLAINTEXT
// Assuming cursor is on line 20 of utils.js
let someFunction = function() {
  if (/* cursor here */) { // Line 20 - Cursor Position
    return true;
  }
  return false;
};

// Show lines around the cursor
let anotherVariable = 'some value';

// Sensitive data is not shown in context.
Dynamic Context:
PLAINTEXT
if (/* cursor here */) { // Line 20 - Cursor Position
  return true;
}

By following these principles, you can ensure that Copilot has the right amount of context to provide useful suggestions without compromising project security.

**Comparison:**

```
File	Context Lines Before	Cursor Line	Context Lines After
utils.js	10	20	5
```

**Code:**

```javascript
// Example function with dynamic contextualization
function someFunction() {
  if (/* cursor here */) { // Line 20 - Cursor Position
    return true;
  }
  return false;
}
```

**Example input / output:**

```
Input: User is editing utils.js and the cursor is on line 20.
Output:
PLAINTEXT
Copy
let someFunction = function() {
  if (/* cursor here */) { // Line 20 - Cursor Position
    return true;
  }
  return false;
};

// Sensitive data is not shown in context.
```

**Data:**

```json
{
  "file": "utils.js",
  "contextLinesBefore": 10,
  "contextLinesAfter": 5,
  "cursorLine": 20,
  "sensitiveDataExcluded": true
}
```

**Flow:**

1. Start
2. Cursor Position?
3. Get Context Lines
4. Filter Sensitive Data
5. Send to AI
6. Receive Suggestions
7. Display Suggestions

