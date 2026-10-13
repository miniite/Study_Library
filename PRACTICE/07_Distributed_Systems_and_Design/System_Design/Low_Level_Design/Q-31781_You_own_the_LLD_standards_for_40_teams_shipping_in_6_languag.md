---
type: "practice"
difficulty: "Expert"
---

# You own the LLD standards for 40 teams shipping in 6 languages; how do you codify class-design and API conventions so they raise the floor without becoming a bureaucratic gate that senior engineers route around?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When designing low-level design (LLD) standards for 40 teams in multiple programming languages, the key is to establish a robust yet flexible framework that balances consistency with agility. This involves defining clear class and API conventions, ensuring they are widely understood, and integrating them into development workflows while maintaining enough flexibility to accommodate diverse team needs.

#### Triaging the Question

The core of this question lies in striking a balance between enforcing standardized practices across multiple teams while respecting the autonomy and expertise of senior engineers. The interviewer is probing how you can create comprehensive guidelines that promote best practices without stifling innovation or becoming an impediment to development.

#### Assumptions
and Context
We have 40 teams distributed across 6 programming languages.
Each team has at least one senior engineer responsible for code quality.
Teams are autonomous in their day-to-day operations but must adhere to the LLD standards.
The goal is to raise the floor of our technical practices without creating bureaucratic overhead.

#### First-Principles Solution

Define Clear Goals: The primary objective is to improve the overall quality and maintainability of codebases while fostering a collaborative culture that values shared knowledge. This can be achieved through:

Establishing a clear set of coding standards.
Defining best practices for API design.
Promoting modular, reusable components.

Language-Specific Guidelines: Given the multiple programming languages involved, it's crucial to create language-specific guidelines but ensure they are compatible with common principles. For instance:

Python: PEP 8 style guide, DRY (Don't Repeat Yourself) principle.
Java: Google Java Style Guide, SOLID principles.
JavaScript: Airbnb.js or ESLint rules.

Modular Components and Reusability: Encourage the development of reusable components to reduce redundancy and improve maintainability. This includes:

Defining a component library that can be shared across teams.
Implementing version control strategies for components.

API Design Principles: Standardize API design practices, focusing on:

#### Cons
istent naming conventions (e.g., snake_case for variables).
Well-documented and versioned APIs.
Error handling best practices.

Tooling and Automation: Leverage tools to enforce coding standards and ensure compliance:

Code linters and formatters (Prettier, ESLint).
Static code analysis tools (SonarQube, Flake8).

Training and Documentation: Provide training sessions and comprehensive documentation to ensure all engineers understand the standards. This includes:

Regular workshops on best practices.
Maintaining up-to-date documentation repositories.

Feedback Loops: Implement a system for continuous feedback and improvement:

Code reviews with constructive feedback.
Regular retrospectives to identify areas of improvement.

Flexibility and Adaptability: Recognize that different teams may have unique requirements. Allow some flexibility but maintain core standards. For example, allow custom implementations as long as they adhere to the defined principles.

#### Closing Summary

By codifying class design and API conventions in a way that respects team autonomy while promoting shared best practices, we can significantly enhance code quality across all 40 teams without creating bureaucratic bottlenecks. The key is to establish clear goals, language-specific guidelines, modular components, standardized API design, effective tooling, comprehensive training, continuous feedback loops, and flexibility.

#### Common Mistakes
and Follow-Ups

Mistake: Overly prescriptive standards that stifle innovation.

#### Follow-up
Encourage senior engineers to contribute to the development of these guidelines to ensure they are practical and realistic.

Mistake: Lack of enforcement mechanisms, leading to inconsistent adherence.

#### Follow-up
Implement automated tools for code quality checks and regular code reviews.

Mistake: Ignoring team-specific needs, leading to resistance and poor adoption.

#### Follow-up
Provide mechanisms for teams to request exceptions or modifications that align with their unique requirements.

**Comparison:**

```
Example of a table summarizing different languages and their specific guidelines: | Language | Guideline | Example | |----------|-------------------------|----------------------------| | Python | PEP8 Style Guide | def greet(self): | | Java | Google Java Style Guide | public User getUser(); | | JavaScript | Airbnb.js | function greet() {} |
```

**Code:**

```python
# Example of a Python class definition following PEP8 guidelines
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def greet(self):
        print(f"Hello, {self.name}!")
```

**Example input / output:**

```
Example input-output dry-run for a simple API endpoint:
JSON
Copy
Input:
POST /users/
{
  "name": "John Doe",
  "email": "john.doe@example.com"
}

Output:
201 Created
{
  "id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

**Data:**

```
Sample API documentation schema:
JSON
Copy
{
  "api": {
    "name": "User",
    "version": "1.0",
    "endpoints": [
      {
        "method": "GET",
        "path": "/users/{userId}",
        "responses": {
          "200": "User object"
        }
      },
      {
        "method": "POST",
        "path": "/users/",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/User"
              }
            }
          }
        },
        "responses": {
          "201": "User object created"
        }
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Code Review Request
4. Review Team
5. Check Against Linting Rules
6. Passes Linting?
7. Successfully Integrated intoCodebase
8. Manual Fixes andResubmission
9. Feedback Loop: Suggestionsfor Improvement

