---
type: "practice"
difficulty: "Intermediate"
---

# Why do many teams review the prompt and config of an LLM feature in a pull request even though the actual model call happens against a hosted API?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Many teams review the prompt and configuration of an LLM feature during the pull request (PR) process because this helps ensure that the intended behavior is correctly specified, maintainability is improved, and security risks are minimized. The code itself, which calls the hosted API, remains unchanged since it's part of the infrastructure.

#### Triage

The question probes why teams rigorously review high-level components like prompts and configurations in PRs even though the actual model calls occur via a hosted API. This involves understanding the importance of version control, maintainability, security, and best practices in software development.

#### First-Principles Solution

Code vs. Configuration: The code that makes the API call is typically immutable or only slightly modified when changes are made to models, APIs, or configurations. Therefore, reviewing these high-level components directly ensures that any issues related to prompt design, data handling, and model usage are caught early.

Version Control: By including these elements in the codebase, teams can manage versions more effectively. This is particularly useful when multiple prompts or configurations are used across different features or experiments. Version control systems like Git allow for easy tracking of changes over time.

Maintainability and Collaboration:

Documentation: Keeping prompt and configuration details within the codebase enhances documentation, making it easier for new team members to understand how the system works.

#### Cons
istency: Ensures that all team members use consistent prompts and configurations, reducing errors and improving collaboration.

Security and Compliance:

Sensitive Data Handling: Prompts might contain sensitive information or follow specific security protocols. Reviewing them ensures these guidelines are followed consistently.
Regulatory Requirements: Some industries have strict regulations regarding how data is handled and processed. Including prompt configurations in the codebase helps ensure compliance.

Debugging and Testing:

Reproducibility: Keeping prompts and configurations close to the model calls allows for easier debugging and testing. It’s simpler to trace back issues from the point of failure.
Automated Testing: Integration tests can be written against specific prompts and configurations, ensuring that changes do not break existing functionality.

#### Closing Summary

In summary, reviewing prompts and configurations in PRs is a best practice that enhances code quality, maintainability, security, and compliance. While the actual model calls are made via a hosted API, keeping these details within the codebase provides numerous benefits.

#### Common Mistakes/Follow-Ups
Ignoring Prompts and Configurations: Teams might overlook reviewing prompts and configurations, thinking they are less critical than other parts of the code.
Over-Engineering: Conversely, teams might overcomplicate their approaches by unnecessarily re-implementing API calls in the codebase when simpler solutions like documentation or external configuration files suffice.

Reviewing these elements is a strategic decision that supports overall project health and robustness.

