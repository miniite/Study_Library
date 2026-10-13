---
type: "practice"
difficulty: "Deep"
---

# How does dvc repro decide which stages to rerun, and why does it depend on Git-tracked dependencies and outputs?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Data with DVC](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

DVC (Data Version Control) uses a Directed Acyclic Graph (DAG) to track data dependencies and decides which stages need to be rerun based on changes in tracked files. It relies heavily on Git's dependency graph to determine the minimal set of actions required when you update your DVC project.

#### Triage

The question is probing into the internal workings of DVC and its dependency management mechanism. Specifically, it aims to understand how DVC decides which stages (data processing steps) should be rerun based on changes in Git-tracked dependencies and outputs. This involves explaining both the theoretical concept and practical implementation details.

First Principles

DVC uses a DAG-based approach to manage data dependencies in machine learning projects. It tracks all data, code, and model files using Git. The key idea is that DVC ensures reproducibility by maintaining a history of transformations applied to data, similar to how Git maintains the history of code changes.

Step-by-Step Solution

Project Structure:

A DVC project contains three main directories: dvc.yaml, .gitignore.dvc, and possibly other subdirectories.

DAG Construction:

When you run a command like dvc repro or dvc run, DVC constructs a DAG that represents the dependencies between data, code, and output files.
This DAG is built by analyzing dvc.yaml files, which define stages (data processing steps) and their inputs and outputs.

Tracking Files:

DVC uses Git to track all project files. When you add a file to .gitignore.dvc, it indicates that the file should be managed by both Git and DVC.
Git-tracked dependencies include any input data or code that needs to be processed, as well as output files generated from those inputs.

Change Detection:

When dvc repro is run, DVC checks for changes in Git-tracked dependencies using Git’s diff functionality.
If a tracked file has changed, DVC uses the DAG to determine which stages need to be rerun. For example, if a dataset (input) changes, all stages that depend on this dataset will need to be updated.

Minimal Rerun:

DVC aims to rerun only the minimal set of stages necessary to produce the latest outputs. This is achieved by traversing the DAG and identifying the nodes affected by the changes.
For instance, if a preprocessing script (input) changes but its output hasn't been modified directly, other dependent stages can reuse the cached output.

#### Closing Summary

In summary, DVC leverages Git’s dependency management capabilities to track data dependencies. By constructing a DAG and using Git’s diff functionality, it efficiently determines which stages need to be rerun when there are changes in tracked files. This ensures that only necessary steps are executed, maintaining both efficiency and reproducibility.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the importance of Git-tracked dependencies. Users might think DVC can handle all file tracking without explicit Git configuration.

#### Follow-Up
How does DVC handle large datasets or distributed systems? What are some best practices for managing data in a DVC project?

**Comparison:**

```
Stage	Input Files	Output Files
1	raw_data.csv	processed_data.csv
```

**Code:**

```python
# Example code to illustrate how DVC tracks dependencies and reruns stages

import dvc.api

def run_dvc_repro():
    # Define the stage with input and output files
    dvc.api.run(
        cmd='echo "Data preprocessing" > processed_data.csv',
        out=['processed_data.csv'],
        deps=['raw_data.csv']
    )

    # Simulate a change in raw data file
    open('raw_data.csv', 'a').write('\nnew data')

    # Re-run DVC to detect and rerun necessary stages
    dvc.api.repro(all_outs=True)
```

**Example input / output:**

```text
Input:
- `raw_data.csv` (unchanged)

Action:
- Add new data to `raw_data.csv`

Output:
- Run DVC repro command.
- Only the `process_data` stage is rerun, producing a new `processed_data.csv`.
```

**Data:**

```yaml
# Example dvc.yaml stage definition
stages:
  - name: process_data
    cmd: echo "Data preprocessing" > processed_data.csv
    out:
      - processed_data.csv
    deps:
      - raw_data.csv
```

**Flow:**

1. Git-tracked dependencies
2. DAG construction
3. Change detection in Git
4. Rerun minimal set of stages

