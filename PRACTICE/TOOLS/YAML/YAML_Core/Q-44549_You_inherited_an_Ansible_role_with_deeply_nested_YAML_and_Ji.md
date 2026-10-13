---
type: "practice"
difficulty: "Intermediate"
---

# You inherited an Ansible role with deeply nested YAML and Jinja templating inside string values. A play fails only on some hosts. How do you distinguish a YAML parse issue from a Jinja evaluation issue?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When troubleshooting an Ansible playbook that fails sporadically, distinguishing between YAML parsing issues and Jinja evaluation issues can be challenging but critical. By carefully examining error messages, understanding the nature of the failures, and applying a structured debugging process, you can identify and resolve these problems.

#### Triaging the Question

The question revolves around diagnosing why an Ansible playbook fails on some hosts while succeeding on others. The complexity arises from deeply nested YAML structures combined with Jinja templating in string values, making it hard to pinpoint the exact issue. The key is understanding what constitutes a YAML parse error versus a Jinja evaluation error.

#### First-Principles Solution

Understanding the Error Messages:

Ansible provides detailed error messages that indicate whether the failure occurred during the parsing of the YAML file or while evaluating Jinja templates.
YAML Parse Errors: These are typically straightforward and occur before any processing of the playbook starts. They might appear in the Ansible output as "failed to parse" errors, followed by a stack trace pointing to the problematic line.
Jinja Evaluation Errors: These happen after the YAML has been parsed successfully but during the execution phase when Jinja templating is applied. The error messages usually include context from where the Jinja code was evaluated.

Reviewing the Playbook Structure:

Examine the deeply nested YAML structure for any syntax errors or misplaced delimiters (e.g., commas, quotes). Ensure that variables are correctly defined and used within Jinja templates.
Use debug modules to print out values of complex variables before they enter Jinja templates to ensure their state is as expected.

Isolating the Issue:

Temporarily simplify the problematic section by removing nested structures or Jinja templates, then gradually reintroduce them until the failure reoccurs.
Utilize Ansible’s debug module to log intermediate values and understand how variables evolve during execution.

Debugging Strategies:

Check Variable Definitions: Ensure all required variables are defined and correctly passed to tasks. Missing or undefined variables can lead to Jinja evaluation failures.
Validate Jinja Templates: Use tools like jinja2 in Python to validate templates independently before integrating them into Ansible playbooks.

Testing Across Different Hosts:

Run the playbook on multiple hosts with varying environments to identify patterns or commonalities among hosts that fail. This can help narrow down if it’s a specific environment issue or a generic configuration problem.
Use --check mode to simulate tasks without modifying real systems and observe how different parts of the play behave.

#### Closing Summary

By methodically examining error messages, structuring your troubleshooting approach, and utilizing debug strategies, you can effectively distinguish between YAML parse issues and Jinja evaluation issues in Ansible playbooks. Properly handling these errors is crucial for maintaining robust and maintainable automation scripts.

#### Common Mistakes/Follow-Ups
Overlooking Variable Scope: Ensuring that variables are correctly scoped and available throughout the playbook.
Misinterpreting Error Messages: Misunderstanding the difference between parse and evaluation errors can lead to misdiagnosis.
Skipping Debugging Steps: Relying solely on error messages without further investigation might skip over subtle issues.

By following these steps, you can ensure that your Ansible playbooks are reliable and easy to debug, even in complex scenarios involving nested YAML and Jinja templates.

