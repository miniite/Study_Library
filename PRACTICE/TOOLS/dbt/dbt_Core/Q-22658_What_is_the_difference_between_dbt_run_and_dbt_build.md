---
type: "practice"
difficulty: "Basic"
---

# What is the difference between dbt run and dbt build?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a nutshell, dbt run and dbt build are both commands used in dbt to compile and execute your models, but they serve slightly different purposes. Let's break down their differences from first principles.

#### Triage

The question is asking us to differentiate between two dbt commands: run and build. The interviewer might be probing our understanding of the workflow steps involved and how these commands interact with each other and the overall build process in dbt. We need to assume a basic dbt setup, where we have models written using SQL or a DSL (domain-specific language) like dbt’s templating language.

#### First-Principles Solution
dbt Build: This command compiles all your dbt files into a target directory, which includes .sql files for each model along with their dependencies.
dbt Run: This command executes the compiled SQL code in the target directory and populates the database with the results.
Detailed Explanation
dbt Build
Purpose: Compilation of dbt files into a target directory.
Process:
dbt parses all your models, macros, seeds, etc., and generates .sql files in the target/compiled folder.
It also writes dependency graphs (manifest.json) to help with execution order.
Output: A set of compiled SQL scripts ready for execution.
dbt Run
Purpose: Execution of models using the compiled SQL code.
Process:
Uses the target/compiled directory as input.
Executes each model in dependency order based on the dependency graph defined by manifest.json.
Populates the database with the results of these model executions.
Output: A populated database schema with data from your models.

#### Closing Summary

In summary, dbt build is about generating and organizing the SQL code you need to run, while dbt run actually executes this generated code. Understanding their roles in the dbt workflow helps streamline development and ensure that your models are correctly implemented and deployed.

#### Common Mistakes/Follow-Ups
Mistake: Confusing these commands with each other; both do not directly interact with the database, but dbt run does.

#### Follow-Up
Can ask about additional dbt commands like seed, test, or debug to explore more of the tool's capabilities.

**Code:**

```python
# Example of a simple dbt command in a shell prompt
dbt build  # Compiles models into target directory

dbt run     # Executes the compiled SQL code
```

