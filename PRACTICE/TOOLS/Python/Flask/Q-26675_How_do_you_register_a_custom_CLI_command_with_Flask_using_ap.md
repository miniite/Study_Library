---
type: "practice"
difficulty: "Intermediate"
---

# How do you register a custom CLI command with Flask using @app.cli.command()?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Flask vs Dedicated Model Servers](<../../../../TOOLS/Python/Flask/Flask_vs_Dedicated_Model_Servers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To register a custom CLI command in Flask, we use the @app.cli.command() decorator to define our function as a command that can be run from the terminal. This is a straightforward way to extend your Flask application's functionality by adding commands for tasks like database migrations or seeding sample data.

The core idea here is to understand how Flask integrates with its CLI system and how you can leverage this integration to add custom commands easily. We'll start by triaging the question, then walk through the steps from first principles.

