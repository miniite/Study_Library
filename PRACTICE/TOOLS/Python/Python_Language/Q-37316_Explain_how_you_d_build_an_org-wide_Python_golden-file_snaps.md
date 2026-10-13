---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd build an org-wide Python golden-file/snapshot testing standard so a feature-engineering change can't silently alter existing features for any model that depends on it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Drift-triggered Retraining](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistency and prevent unintended changes in feature engineering across our organization, we need to establish an org-wide Python golden-file/snapshot testing standard. This system will capture snapshots of critical features at regular intervals and compare them with new data to detect any discrepancies.

First, let's define what a "golden file" is: it’s a snapshot of the desired state or expected output that we want our feature engineering pipeline to produce. By comparing this golden file against current outputs, we can catch any unexpected changes early in the development process.

#### Triaging the Question

The core of the question is about ensuring the reproducibility and consistency of our feature engineering processes across teams. The key areas being probed are:

Data Management: How do you manage and version your datasets?
Automation and CI/CD: Can we automate this process to run continuously or on a schedule?
Error Detection and Prevention: What kind of mechanisms can be put in place to detect and prevent silent changes?

#### Assumptions
We are using Python for feature engineering.
There is an existing feature engineering pipeline that processes raw data into features used by various models.
The organization has multiple teams and projects that depend on these features.

#### First-Principles Solution

Define Key Features: Identify which features are critical for the models in use. These should be captured as golden files.

Data Collection and Versioning:

Collect raw data from the source.
Process this data through our feature engineering pipeline to generate the current state of features.
Store these outputs (golden files) in a version-controlled system like Git or a dedicated storage solution with timestamps.

Automated Pipeline for Golden Files:

Implement an automated pipeline that runs on a schedule or after each change in the feature engineering code.
This pipeline should generate new golden files and compare them with existing ones using a comparison tool (e.g., diff).

Golden File Storage and Comparison Tool:

Use tools like deepdiff, csv_diff, or custom scripts to detect differences between old and new golden files.
Store these comparison results in a database or logging system for easy auditing.

Notification Mechanism:

If any discrepancies are found, the system should alert relevant teams (e.g., via Slack, email) with details of which feature has changed.
Implement a rollback process to revert changes if necessary.

Continuous Integration/Continuous Deployment (CI/CD):

Integrate this pipeline into our existing CI/CD workflow so that every change in the code triggers a re-generation and validation of golden files.
Detailed Implementation

Feature Selection:

Use a script to select critical features based on model dependencies.
PYTHON
import pandas as pd

def select_critical_features(model_dependencies):
    # Logic to determine which columns are used by models
    return list(model_dependencies.keys())

critical_features = select_critical_features(model_dependencies)

Data Processing Pipeline:

Write a Python script that processes raw data and generates feature files.
PYTHON
def process_data(raw_data_path, processed_data_path):
    # Load and preprocess raw data
    df = pd.read_csv(raw_data_path)

    # Apply feature engineering transformations
    transformed_df = transform_features(df)

    # Save the transformed features to a file
    transformed_df.to_csv(processed_data_path, index=False)

Golden File Generation:

Implement logic to generate and save golden files.
PYTHON
def create_golden_file(current_features):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    golden_file_path = f"golden_files/{timestamp}.csv"
    current_features.to_csv(golden_file_path, index=False)
    return golden_file_path

Golden File Comparison:

Write a comparison tool to detect differences between golden files.
PYTHON
from deepdiff import DeepDiff

def compare_golden_files(old_golden_file, new_golden_file):
    old_df = pd.read_csv(old_golden_file)
    new_df = pd.read_csv(new_golden_file)

    diff = DeepDiff(old_df, new_df, significant_digits=6)
    return diff

Alerting Mechanism:

Set up a mechanism to alert teams if differences are detected.
PYTHON
import smtplib

def send_alert(diff):
    if diff:
        # Send an email or Slack message with details of the change
        server = smtplib.SMTP('smtp.example.com', 587)
        server.starttls()
        message = f"Feature {diff} has changed."
        server.sendmail("from@example.com", "to@example.com", message)
        server.quit()

#### Closing Summary
and Common Mistakes

In summary, we need a robust system that can automatically generate and compare golden files to ensure the consistency of feature engineering across our organization. This involves identifying critical features, automating data processing pipelines, comparing outputs using powerful tools, and setting up alert mechanisms.

#### Common mistakes
include:

Failing to define which features are critical.
Not integrating this process into CI/CD workflows.
Overlooking the importance of version control and auditing.
Ignoring edge cases where feature dependencies might change silently.

By addressing these points and implementing a well-thought-out solution, we can ensure that our models remain consistent and reliable over time.

**Comparison:**

```
Feature	Model Dependencies
feature1	model1
feature2	model1
feature3	model2
feature4	model2
```

**Code:**

```python
import pandas as pd
from datetime import datetime

def select_critical_features(model_dependencies):
    # Logic to determine which columns are used by models
    return list(model_dependencies.keys())

def process_data(raw_data_path, processed_data_path):
    df = pd.read_csv(raw_data_path)

    transformed_df = transform_features(df)

    transformed_df.to_csv(processed_data_path, index=False)

def create_golden_file(current_features):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    golden_file_path = f"golden_files/{timestamp}.csv"
    current_features.to_csv(golden_file_path, index=False)
    return golden_file_path

def compare_golden_files(old_golden_file, new_golden_file):
    old_df = pd.read_csv(old_golden_file)
    new_df = pd.read_csv(new_golden_file)

    diff = DeepDiff(old_df, new_df, significant_digits=6)
    return diff
```

**Example input / output:**

```

Input:

PYTHON
Copy
raw_data_path = "data/raw_data.csv"
processed_data_path = "data/processed_data.csv"

# Run the processing pipeline
process_data(raw_data_path, processed_data_path)

Output:

processed_data.csv containing transformed features.

Golden File Creation:

PYTHON
Copy
critical_features = select_critical_features(model_dependencies)
golden_file_path = create_golden_file(pd.DataFrame(critical_features))

Comparison:

PYTHON
Copy
old_golden_file = "golden_files/20231015_1430.csv"
new_golden_file = "golden_files/20231016_1500.csv"

diff = compare_golden_files(old_golden_file, new_golden_file)

Alert:

PYTHON
Copy
if diff:
    send_alert(diff)
```

**Data:**

```yaml
model_dependencies:
  - model1: [feature1, feature2]
  - model2: [feature3, feature4]
```

**Flow:**

1. Raw Data
2. Feature Engineering
3. Processed Features
4. Golden File Storage
5. Golden Files Comparison
6. Alerts & Notifications

