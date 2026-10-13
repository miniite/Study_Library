---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: you introduced a shared "utils" or "common" library everyone edits and it's now a release bottleneck across the org; what's your criterion for what becomes a shared library vs copied code, and how do you decompose the god-module?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a shared "utils" or "common" library that has become a bottleneck, it’s crucial to balance the benefits of reuse against the costs of maintenance and release cycles. The key is to define clear criteria for what constitutes a shared utility versus code that can be safely copied, and then implement a decommissioning strategy for monolithic libraries.

#### Triage
Understanding the Problem

The interviewer is probing your approach to modularization, how you ensure efficient collaboration across teams, and your strategies for managing dependencies. The core of this question revolves around defining the boundaries between shared utilities and local codebases while ensuring that the library remains performant and maintainable.

#### First-Principles Solution
Defining Shared Utilities
Scope of Reuse: Determine if the utility is broadly applicable across multiple teams or projects. If it’s a general-purpose function like logging, caching, or type conversion, it likely should be shared.
Frequency of Change: Consider how often the functionality changes. Frequent updates may make local copies more practical to reduce integration issues.
Complexity and Size: Evaluate the complexity and size of the code. Complex utilities that affect many aspects of an application should be in a shared library, while simpler ones can be copied.
Step-by-Step Solution
Identify Common Patterns: Look for repeated patterns or functions across projects. If multiple teams are implementing similar functionality, it’s likely a good candidate for a shared library.
Define Clear Contracts: Establish clear interfaces and contracts for the utilities. This helps maintain consistency and reduces coupling issues.
Automate Testing: Ensure comprehensive testing of shared libraries to catch issues early in integration phases.
Release Management: Implement a robust release management process to minimize disruption when changes are made.
Decompose Monolithic Libraries: Break down large, monolithic utilities into smaller, more focused components.
Decomposing the God-Module
Identify Overlapping Functions: Analyze which functions are used in isolation versus those that rely on others. Separate them into distinct modules or files.
Refactor Based on Usage: Refactor based on how different parts of the codebase use the library. Move shared functionalities to a central location while keeping local, project-specific implementations.
Implement Versioning and Branching: Use version control and branching strategies to manage changes without breaking existing integrations.

#### Closing Summary

By defining clear criteria for what becomes a shared utility versus copied code, and implementing a decommissioning strategy for monolithic libraries, you can ensure efficient collaboration while maintaining performance and reducing release bottlenecks. It’s important to regularly review and refactor the shared library to keep it lean and maintainable.

#### Common Mistakes
and Follow-Ups
Over-Engineering: Overly complex or rigid design can lead to maintenance issues.
Ignoring Local Variations: Failing to account for project-specific variations can cause unnecessary duplication.
Lack of Automation: Manual testing and integration can introduce bugs that automated processes would catch.

Would you like me to elaborate on any specific aspect, such as release management tools or implementation strategies?

**Comparison:**

```
Criteria	Definition
Scope of Reuse	A utility should be shared if it is used across multiple projects.
Frequency of Change	Utilities that change infrequently are good candidates for sharing.
Complexity and Size	Large, complex utilities with many dependencies should be shared.
```

**Code:**

```python
# Example of a simple utility function in a shared library
def format_log_message(message, level):
    return f"[{level}] {message}"
```

**Example input / output:**

```text
Input:
log_message = format_log_message("Error occurred", "ERROR")

Output:
[ERROR] Error occurred
```

**Data:**

```json
{
  "library": {
    "name": "shared-utils",
    "version": "1.0.2"
  },
  "dependencies": [
    { "name": "common-logger", "version": "3.5.1" }
  ]
}
```

**Flow:**

1. Identify Common Patterns
2. Define Clear Contracts
3. Automate Testing
4. Release Management
5. Decompose MonolithicLibraries

