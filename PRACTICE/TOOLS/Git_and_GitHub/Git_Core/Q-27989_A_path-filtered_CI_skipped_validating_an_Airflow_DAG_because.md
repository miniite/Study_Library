---
type: "practice"
difficulty: "Deep"
---

# A path-filtered CI skipped validating an Airflow DAG because only a shared operators library changed, not the DAG's own file, and a broken import shipped to the scheduler. How do you make change detection dependency-aware so it never skips a DAG it should have re-validated?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To ensure that CI properly detects changes in dependencies and re-validates affected Airflow DAGs, we need to implement a more sophisticated change detection mechanism that tracks import statements within DAG files. This involves parsing the code to identify imported modules and comparing them against the shared operators library.

#### Triaging the Question

The core issue here is that the path-filtered CI skipped validating an Airflow DAG because only the shared operators library changed, not the DAG's own file, leading to a broken import in the scheduler. To address this, we need to make our change detection mechanism dependency-aware, ensuring it re-validates any DAGs with potential import issues.

#### Assumptions
The CI system uses path filtering to avoid unnecessary validations.
Changes are detected based on file paths.
Airflow DAG files contain import statements that reference shared operators or other modules.
The shared operators library is version-controlled in a Git repository (e.g., GitHub).

#### First-Principles Solution
Identify Import Statements: Develop a script or tool to parse the content of each DAG file and extract all import statements.
Track Dependencies: Maintain a database or list that maps imported modules to their corresponding files in the shared operators library.
Change Detection Logic:
Monitor changes to both DAG files and shared operators library files.
When a change is detected, determine if it affects any DAG by comparing the updated import statements with the tracked dependencies.
If an affected DAG is found, trigger its re-validation in CI.
Step-by-Step Implementation

Parser Development:

PYTHON
# Example parser function to extract imports from a DAG file
def parse_dag_imports(file_path):
    with open(file_path) as file:
        content = file.read()
        import_statements = re.findall(r'from\s+(\w+)\s+import', content)
        return {stmt: file_path for stmt in import_statements}

Dependency Mapping Database:

YAML
# Example YAML structure of dependency mapping
dependencies:
  operators_a: /path/to/operators/a.py
  operators_b: /path/to/operators/b.py

Change Detection and Validation Triggering Logic:

PYTHON
def detect_changes(dag_file_changes, operator_library_changes):
    affected_dags = set()
    for dag_change in dag_file_changes:
        imports = parse_dag_imports(dag_change['file_path'])
        for module_name, file_path in imports.items():
            if file_path in operator_library_changes:
                affected_dags.add(os.path.basename(file_path))

    # Trigger re-validation for affected DAGs
    for d in affected_dags:
        trigger_validation(d)

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a dependency-aware change detection mechanism, we ensure that CI always validates Airflow DAGs when dependencies are modified. This approach requires careful handling of import statements and effective tracking of changes to both the DAG files and shared libraries.

#### Common Mistakes

Overlooking edge cases where imports might be dynamic or involve complex dependencies.
Failing to account for local vs global import contexts, which can lead to false positives/negatives in dependency detection.
Not integrating seamlessly with existing CI pipelines, potentially causing performance issues due to frequent re-validations.

#### Follow-Ups

Discussing potential integration with Git hooks or webhooks to monitor changes more closely.
Exploring automated testing strategies for shared libraries and how they impact downstream DAGs.

#### Cons
idering the trade-offs between accuracy and system overhead in implementing a robust dependency-aware CI mechanism.

