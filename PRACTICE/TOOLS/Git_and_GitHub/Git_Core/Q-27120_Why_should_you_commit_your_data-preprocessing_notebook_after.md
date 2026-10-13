---
type: "practice"
difficulty: "Intermediate"
---

# Why should you commit your data-preprocessing notebook after exporting it to a plain .py script, and how does jupytext help?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as an ML engineer, committing changes is crucial for maintaining a version-controlled environment. Committing a data preprocessing notebook after converting it to a .py script helps in tracking modifications and ensuring reproducibility of your workflows. This practice is especially important when collaborating with others or when using continuous integration/continuous deployment (CI/CD) pipelines.

#### Triage

The question asks two main points:

Why committing data-preprocessing notebooks after exporting them to plain .py scripts is beneficial.
How jupytext aids in managing these transitions smoothly and efficiently.

#### Assumptions
You are working on an ML project that uses Jupyter Notebooks for preprocessing steps before training models.
The goal is to maintain a clean, version-controlled repository with clear separation of concerns between interactive development and automated scripts.
Step-by-Step Solution

Version Control Benefits:

Tracking Changes: Committing the notebook allows you to track modifications over time, making it easier to understand what changes were made and when.
Reproducibility: Keeping a record of preprocessing steps ensures that others can replicate your data transformations if needed.
Collaboration: When working in teams, version control helps manage conflicts and updates efficiently.

Preprocessing Workflow:

Interactive Development: Jupyter Notebooks provide an interactive environment for experimenting with code snippets, visualizations, and data manipulations.
Automated Scripts: Once the preprocessing steps are finalized, they can be converted to plain .py scripts for production or integration into a larger pipeline.

Role of jupytext:

File Conversion: jupytext is a tool that enables seamless conversion between Jupyter Notebook (ipynb) and Python script formats.

#### Cons
istency: It ensures that the content in both formats remains consistent, reducing the risk of divergent versions.
Automation: It simplifies the process of moving code from an interactive notebook to a production-ready script.

#### Example
Workflow

Initial Development:

PYTHON
# Initial data preprocessing steps
import pandas as pd

df = pd.read_csv('data.csv')
cleaned_df = df.dropna()

# Plotting some initial insights
%matplotlib inline
import matplotlib.pyplot as plt
cleaned_df.plot(kind='line', x='date', y='value')

Finalizing and Exporting:

Convert the notebook to a .py script using jupytext.
BASH
jupytext --to py preprocessing.ipynb

Committing Changes:

MARKDOWN
Commit message: "Refined data cleaning pipeline and visualizations."
git add preprocessing.ipynb
git commit -m "Refined data cleaning pipeline and visualizations."

Automation with jupytext:

Use a CI/CD pipeline to automatically convert notebooks to scripts during deployments.
YAML
jobs:
  build:
    steps:
      - run: jupytext --to py preprocessing.ipynb

#### Closing Summary
and Common Mistakes
Committing is crucial: Failing to commit changes can lead to lost work and make collaboration difficult.
Use jupytext effectively: Properly configuring jupytext ensures that both notebook and script versions are kept in sync, reducing inconsistencies.
Common Follow-Ups
What are the advantages of keeping code in a Jupyter Notebook?
How does using jupytext impact performance during model training?
Can you provide an example of how jupytext can be integrated into a Git workflow?

**Comparison:**

```
Step	Action
1	Convert Jupyter notebook to .py script.
2	Commit the changes in Git.
3	Integrate jupytext into CI/CD pipeline.
```

**Code:**

```python
# Example Python script after exporting from Jupyter notebook
import pandas as pd

def preprocess_data(file_path):
    df = pd.read_csv(file_path)
    cleaned_df = df.dropna()
    return cleaned_df
```

**Example input / output:**

```

Initial Jupyter Notebook content:

PYTHON
Copy
import pandas as pd

df = pd.read_csv('data.csv')
cleaned_df = df.dropna()

%matplotlib inline
import matplotlib.pyplot as plt
cleaned_df.plot(kind='line', x='date', y='value')

Converted .py script using jupytext:

PYTHON
Copy
# preprocessing.py
import pandas as pd

def preprocess_data(file_path):
    df = pd.read_csv(file_path)
    cleaned_df = df.dropna()
    return cleaned_df
```

**Data:**

```text
- Sample data: `data.csv` containing raw data.
```

**Flow:**

1. Initial Jupyter Notebook
2. Convert using jupytext --to pypreprocessing.ipynb
3. Commit to Git repository
4. Integrate with CI/CD forautomated conversion
5. Production-ready scriptdeployment

