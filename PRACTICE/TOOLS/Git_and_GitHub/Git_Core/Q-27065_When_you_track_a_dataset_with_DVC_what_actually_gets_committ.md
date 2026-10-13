---
type: "practice"
difficulty: "Deep"
---

# When you track a dataset with DVC, what actually gets committed to Git versus what is stored in remote storage like S3?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Data with DVC](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to understand how DVC (Data Version Control) works alongside Git and GitHub. Essentially, when you use DVC with Git, you're managing the versioning of your dataset in a way that is different from traditional code. When you commit changes using dvc add, DVC stages the data for tracking and Git stores metadata about these staged files.

#### Triaging the Question

The core question here is to understand what is committed to Git versus what is stored remotely, such as S3. The interviewer might be probing your understanding of how DVC handles datasets in a way that integrates seamlessly with Git workflows while keeping data storage separate and efficient.

#### First Principles Solution
Data Tracking: When you use dvc add on a dataset, DVC tracks the file/directory, not its contents. This means the metadata about this file is committed to Git.
Remote Storage: The actual dataset remains stored in remote storage like S3. DVC handles this by storing references (like URIs) and checksums of files in your local .dvc directory.
Detailed Solution

Step 1: Adding a Dataset

When you run dvc add <path_to_dataset>, DVC creates a corresponding dvc.yaml file that contains the metadata about the dataset, including its path and checksum.
This dvc.yaml is then added to your Git repository along with any other code or files.

Step 2: Committing Changes

When you commit changes using git add .dvc/ && git commit, only the DVC metadata (e.g., paths, checksums) are tracked by Git.
The actual data remains untouched and is stored in remote storage like S3. Your .dvc directory contains references to these locations.

Step 3: Pulling and Pushing

When you pull a repository with DVC, the metadata is downloaded from GitHub into your local environment.
To ensure consistency between local and remote environments, you might need to sync datasets using dvc pull.

#### Closing Summary

In summary, DVC commits only the metadata about the dataset (e.g., file paths, checksums) to Git. The actual data remains stored in remote storage like S3. This approach allows for efficient versioning of large datasets while keeping them off your local machine and reducing unnecessary bandwidth usage.

#### Common Mistakes/Follow-Ups
Mistake: Confusing DVC with direct file management. DVC does not manage the files directly but tracks their metadata.

#### Follow-Up
The interviewer might ask about the implications of committing large checksums or if there are better practices for very large datasets.

**Comparison:**

```
Metadata	Description
path	The path to the dataset file/directory.
md5	A checksum of the dataset.
cache	The location in S3 where the dataset is stored (if applicable).
```

**Code:**

```python
# Example of dvc add command
!dvc add dataset.csv

# This will create a .dvc directory with metadata in dvc.yaml
```

**Flow:**

1. Add Dataset
2. dvc add <path>
3. DVC tracks metadata
4. Commit changes to Git
5. Pull datasets from remotestorage using dvc pull

