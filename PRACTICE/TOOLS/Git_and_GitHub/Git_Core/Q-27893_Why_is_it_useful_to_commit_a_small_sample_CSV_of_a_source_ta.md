---
type: "practice"
difficulty: "Basic"
---

# Why is it useful to commit a small sample CSV of a source table for local testing while keeping the full multi-gigabyte extract out of the repo?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gitignore](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitignore.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with large datasets, such as multi-gigabyte CSV files, it's crucial to have a lightweight and efficient way to test your ETL processes on sample data. Committing a small sample CSV of the source table in Git can serve this purpose well while keeping the full dataset out of the repository. This approach leverages the benefits of version control without bloating the repo size.

#### Triage
What's Being Probed

The question is probing your understanding of:

Git and GitHub usage: How they facilitate code and data management.
Data Management Practices: Best practices for handling large datasets in version control systems.
Testing Efficiency: The importance of having a quick and reliable testing environment.

#### Assumptions
You are familiar with basic Git commands.
The source table is likely large, perhaps several gigabytes or more.
Your local development environment needs to test ETL processes on this data.

#### First-Principles Solution
Why Sample CSVs?
Version Control: Small sample files can be committed and tracked in Git efficiently without impacting the repository's size significantly.
Testing Efficiency: Local testing with small datasets is faster, reducing waiting time for tests to complete.
Resource Management: Keeping full-sized data out of the repo saves space and avoids unnecessary bandwidth usage during pull requests.
Step-by-Step Solution

Extract Sample Data:

Use a tool like head (Unix) or pandas in Python to extract a small sample from the large CSV file.
PYTHON
import pandas as pd

# Load the full dataset
df = pd.read_csv('full_dataset.csv')

# Extract 1% of the data as a sample
sample_df = df.sample(frac=0.01)

Save Sample to File:

Save the extracted sample to a new CSV file.
PYTHON
sample_df.to_csv('sample_dataset.csv', index=False)

Commit and Push to Git Repository:

Add, commit, and push the small sample file to your Git repository.
SH
git add sample_dataset.csv
git commit -m "Add sample dataset for local testing"
git push origin main

Local Testing with Sample Data:

Use this sample in your ETL processes locally before integrating the full dataset.
PYTHON
# Load and process the sample data
df = pd.read_csv('sample_dataset.csv')
processed_df = transform(df)

#### Closing Summary
+ Common Mistakes/Follow-Ups

Committing a small sample CSV is beneficial because it strikes a balance between maintaining version control efficiency and providing effective testing environments. It helps in quickly identifying and fixing issues without the overhead of large file sizes.

#### Common Mistakes

Failing to use samples, leading to slow and inefficient local tests.
Overcommitting full-sized datasets, which can bloat the repository unnecessarily.
Neglecting proper documentation around sample data usage and management.

#### Follow-Ups

How do you handle different environments (e.g., development vs. production)?
What tools or techniques would you use to ensure data integrity when working with samples?
How do you manage version control for large binary files like images, if applicable?

**Comparison:**

```
Source Table	Sampled Data
full_dataset.csv (10GB)	sample_dataset.csv (100MB)
```

**Code:**

```python
import pandas as pd

# Load the full dataset
df = pd.read_csv('full_dataset.csv')

# Extract 1% of the data as a sample
sample_df = df.sample(frac=0.01)

# Save the extracted sample to a new CSV file
sample_df.to_csv('sample_dataset.csv', index=False)
```

**Example input / output:**

```text
Input:
- Full dataset path: 'full_dataset.csv'

Output:
- Sample dataset path: 'sample_dataset.csv'
```

**Data:**

```plaintext
full_dataset.csv: A large CSV file with several gigabytes of data.
sample_dataset.csv: A small 1% sample of full_dataset.csv for local testing.
```

**Flow:**

1. Load Full Dataset
2. Filter Sample
3. Save Sample to File
4. Commit and Push to Git Repo
5. Test with Sample Data Locally

