---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle a source that changes its NULL representation over time (empty string one month, literal 'NULL' the next)?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with data from sources that change their NULL representations over time, we need to ensure our preprocessing and cleaning pipelines can handle these changes seamlessly. For example, a source might represent missing values as empty strings one month but switch to using the literal string 'NULL' in subsequent months. This variability requires robust handling to maintain consistency and accuracy.

This question is probing your understanding of:

Data inconsistency: How different sources might use varying NULL representations.
Pipeline flexibility: The ability to adapt preprocessing logic dynamically or robustly.
Error tolerance: Handling unexpected changes without breaking the entire pipeline.

To tackle this, we can follow these steps:

Step 1: Identify and Tag Nullable Representations

First, identify all possible NULL representations used by the source. For our case, both empty strings ('') and literal NULL are candidates.

PYTHON
nullable_representations = ['NULL', '']
Step 2: Clean Data in a Robust Manner

Create a function to clean the data such that it consistently treats these representations as NULL.

PYTHON
def clean_null_representation(data, nullable_reps):
    for rep in nullable_reps:
        data = data.replace(rep, None)
    return data
Step 3: Validate Cleaned Data

Ensure the cleaning process does not introduce new errors by validating a subset of the cleaned data.

PYTHON
clean_data = clean_null_representation(original_data, nullable_representations)
print(clean_data.head())
Step 4: Automate and Log Changes

Automate this process in your ETL pipeline to handle such changes. Also, log these changes for future reference.

PYTHON
def update_pipeline_config(new_nullable_reps):
    # Update configuration files or database entries
    pass

# Example usage:
update_pipeline_config(nullable_representations)

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Failing to identify all NULL representations. Missing a representation can lead to data inconsistencies.
Mistake 2: Not validating the cleaning function thoroughly, which could introduce new errors.

#### Follow-Up
Discuss how you would handle more complex scenarios where different columns might have different NULL representations.

