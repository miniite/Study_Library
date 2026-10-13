---
type: "practice"
difficulty: "Intermediate"
---

# What's your workflow for having the AI generate code while you keep tests running so you catch breakage instantly?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Handling AI Inconsistency](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Handling_AI_Inconsistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using AI-assisted coding tools like Copilot or similar, maintaining a balance between productivity and code quality is crucial. My workflow involves integrating these tools seamlessly with my development environment while keeping automated tests running in the background to catch any potential breakage instantly. This approach ensures that I can leverage the benefits of real-time suggestions without compromising on test coverage.

Firstly, it’s important to understand what the AI-assisted coding tool is capable of and its limitations. For instance, Copilot suggests code snippets based on context but may occasionally make incorrect or suboptimal suggestions. By keeping tests running, we can quickly identify when something goes wrong. Here's a step-by-step breakdown of my workflow:

Setup: Ensure that the AI-assisted coding tool is properly installed and configured in your IDE (Integrated Development Environment) such as VS Code. This involves installing the appropriate extension and customizing settings like enabling or disabling specific suggestions.

Environment Configuration: Configure your development environment to run tests automatically whenever you save a file. Most modern IDEs have built-in support for this, but it can also be done via external tools like GitLab CI/CD pipelines if needed.

Integrating Copilot: Start using the AI-assisted coding tool by enabling it in your editor settings. As you type code, Copilot will provide suggestions and complete lines of code. Be mindful of these suggestions; sometimes they might not align with your project's conventions or requirements.

Running Tests Continuously: Use a testing framework that integrates well with your IDE to run tests automatically. For example, Jest for JavaScript projects or Pytest for Python projects can be configured to run on save. These tools will catch any issues introduced by Copilot’s suggestions almost immediately.

Monitoring and Validation: Keep an eye on the test results as you integrate new code. If a red light appears in your test runner (indicating a failed test), investigate the cause quickly. It could be due to a misalignment between the AI's suggestion and existing logic, or it might indicate a bug that needs fixing.

Refactoring and Cleanup: After integrating suggestions from Copilot, review the changes carefully. Refactor if necessary to maintain code quality and ensure readability. This step is crucial to keep your codebase clean and understandable.

Regular Code Reviews: Even with automated testing, it’s beneficial to perform regular code reviews, possibly using static analysis tools like SonarQube or Pylint, to catch any overlooked issues.

By following this workflow, you can harness the power of AI-assisted coding while maintaining a robust safety net provided by continuous integration and testing. This approach ensures that you stay productive without sacrificing the quality of your code.

#### Common mistakes
include relying too heavily on Copilot's suggestions or neglecting to run tests, which can lead to introducing bugs into your codebase. Always remain vigilant and perform thorough reviews post-automation.

**Code:**

```
Example setup in a .vscode/settings.json for VS Code

{ "editor.rulers": [80], "extensions.ignoreRecommendations": false, "copilot.experimentalFlags": ["--enable-pair-programming"], "terminal.integrated.shell.windows": "C:\Program Files\Git\bin\bash.exe" }

CODE
Copy

```json
# Example Jest configuration in jest.config.js for testing on save
module.exports = {
  // Other configurations...
  watchPlugins: [
    'jest-watch-typeahead/babel',
  ],
  testMatch: [
    '**/__tests__/**/*.+(ts|tsx|js)',
    '**/?(*.)+(spec|test).+(ts|tsx|js)'
  ],
}
```

**Flow:**

1. Setup Environment
2. Enable Copilot
3. Integrate Tests
4. Monitor and Validate
5. Refactor Code
6. Regular Reviews

