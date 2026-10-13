---
type: "practice"
difficulty: "Expert"
---

# How would you validate that a PII-masking transform actually masked every path, including nested JSON and free-text fields, before data lands in the lake?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validation Coverage and Quarantine](<../../../../02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a PII-masking transform properly masks all paths, including nested JSON and free-text fields, we need to validate the transformed data comprehensively. This involves both technical validation and an understanding of potential edge cases.

#### Triaging the Question

The interviewer is probing our ability to design a robust validation framework, handle complex data structures, and consider various types of PII (Personally Identifiable Information). We are expected to reason through multiple steps: identifying what constitutes valid masking, designing test cases, and implementing the checks. Key assumptions include that we have access to both raw and transformed data sets for comparison.

#### First-Principles Solution
Step 1: Define Valid Masking Criteria

For a mask to be considered effective, it must replace sensitive information with a consistent, non-reversible token. For example, phone numbers could become "XXX-XX-XXXX", emails might become "xxxxxx@yyyyyy", and full names could be replaced with "NAME_REDACTED".

Step 2: Extract Data from Raw and Transformed Sets

We will compare the raw data set (RDS) against the transformed data set (TDS). Our goal is to ensure that every field in TDS that corresponds to a PII field in RDS has been masked.

Step 3: Handle Nested JSON Structures

JSON structures need special attention. We must traverse these recursively, checking each key and value for PII content. If a nested object contains sensitive information, it should be fully masked.

Step 4: Process Free-Text Fields

Free-text fields can contain PII in various formats. Regular expressions (regex) or custom parsing rules will help identify potential PII patterns within free text.

#### Implementation
Steps
Setup Environment: We need access to both RDS and TDS, along with a set of valid masking criteria.
Data Preparation:
Load the raw data into a structured format (e.g., DataFrame).
Identify columns containing PII through metadata or schema analysis.
Masking Validation Function:
PYTHON
import re

def is_masked(value, mask_types):
    if not value: return True  # Empty values are considered masked
    for mask_type in mask_types:
        if mask_type['pattern'].search(value):
            return True
    return False

def validate_masking(raw_data, transformed_data, mask_types):
    mismatches = []
    for raw_row, trans_row in zip(raw_data.iterrows(), transformed_data.iterrows()):
        raw_values = raw_row[1]
        trans_values = trans_row[1]
        for col in raw_data.columns:
            if is_masked(trans_values[col], mask_types) != is_masked(raw_values[col], mask_types):
                mismatches.append((raw_values, trans_values))
    return mismatches
Test Cases:
Create a set of test cases covering common PII types and nested structures.
Execution and Reporting:
Run the validation function on both data sets.
Generate reports highlighting any unmatched fields.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By systematically validating masked transformations, we can ensure that sensitive information is fully protected. Key challenges include handling complex nested structures and ensuring comprehensive coverage of potential PII types. Missteps might involve overlooking certain field types or failing to account for edge cases in regular expressions.

#### Common Mistakes

Failing to consider all possible PII formats.
Overlooking recursive traversal in nested JSON fields.
Not validating the masking function thoroughly enough.

For follow-ups, the interviewer may ask about performance optimizations or integrating this validation into a continuous integration pipeline. Understanding these nuances will demonstrate your depth of expertise as a data engineer.

