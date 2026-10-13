---
type: "practice"
difficulty: "Intermediate"
---

# How do you add custom field validation to a Pydantic model in FastAPI, for example checking an email format?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore how to add custom field validation to a Pydantic model in FastAPI, focusing on validating the format of an email address. This involves defining a custom validator function and integrating it with your Pydantic model.

The core idea here is that by leveraging FastAPI's integration with Pydantic, you can easily extend standard data validation capabilities to include custom logic, ensuring that user input meets specific criteria before processing.

FastAPI uses Pydantic models for request body validation. To add a custom email format validator, we'll define a validator function and apply it to the email field in our model.

